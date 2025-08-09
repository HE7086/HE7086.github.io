# Using String as None Type Template Parameters in C++20

C++20 allows using structural types as NTTP, but std::string is not. So here's a Implementation of fixed size string that can be used at compile time.

Deduction guide is also provided for better syntax.

Although `std::copy` is used here, it only happens at compile time. Trivially copyable types can't be moved anyways.

## Implementation
```cpp
template <size_t N, typename CharT = char>
struct fixed_string {
  std::array<CharT, N> data{};

  consteval fixed_string(CharT const (&str)[N]) {
    std::copy_n(str, N, data.begin());
  }
  constexpr CharT const* c_str() const {
    return data.data();
  }
  constexpr std::basic_string_view<CharT> view() const {
    return {data.data(), N - 1};
  }
};
template <size_t N, typename CharT>
fixed_string(const CharT (&)[N]) -> fixed_string<N, CharT>;
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
    std::println("{}", S.view());
  }
};

int main() {
  auto _ = foo<"">{};
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
