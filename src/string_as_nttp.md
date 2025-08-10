# Using String as None Type Template Parameters in C++20

C++20 allows using structural types as NTTP, but std::string is not. So here's a Implementation of fixed size string that can be used at compile time.

Deduction guide is also provided for better syntax.

Although `std::copy` is used here, it only happens at compile time. Trivially copyable types can't be moved anyways.

## Implementation
```cpp
template <size_t N, typename CharT = char>
  requires std::is_nothrow_assignable_v<CharT&, CharT const&>
struct FixedString {
  std::array<CharT, N> data{};

  consteval FixedString(CharT const (&str)[N]) noexcept {
    std::copy_n(str, N, data.begin());
  }
  constexpr CharT const* c_str() const noexcept {
    return data.data();
  }
  constexpr std::basic_string_view<CharT> view() const noexcept {
    return {data.data(), N - 1};
  }
  constexpr operator std::basic_string_view<CharT>() const noexcept {
    return {data.data(), N - 1};
  }
};
template <size_t N, typename CharT>
FixedString(CharT const (&)[N]) -> FixedString<N, CharT>;

template <typename CharT, size_t N>
struct std::formatter<FixedString<N, CharT>, CharT>
    : std::formatter<std::basic_string_view<CharT>, CharT> {

    template <typename FormatContext>
    auto format(FixedString<N, CharT> const& fs, FormatContext& ctx) const {
        return std::formatter<std::basic_string_view<CharT>, CharT>::format(fs, ctx);
    }
};
```

## Usage
* this example requires c++26
```cpp
template <fixed_string S>
class foo {
public:
  foo() {
    std::println("{}", S.c_str());
  }
  ~foo() {
    std::println("{}", S);
  }
};

int main() {
  auto _ = foo<"hello">{};
  auto _ = foo<"world">{};
}
```
* output
```
hello
world
world
hello
```
### Note
* `std::print` does not support different char types yet, e.g. `char8_t`, `char16_t`, etc.
