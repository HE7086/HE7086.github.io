# flake.nix

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
    let
      pkgs = import nixpkgs {
        inherit system;
        config.allowUnfree = true;
      };
    in rec {
      packages.default = pkgs.stdenv.mkDerivation {
        name = "something";
        src = self;
        nativeBuildInputs = with pkgs; [ cmake ];
        buildInputs = with pkgs; [ fmt ];
      };

      devShell.default = pkgs.mkShell {
        inputsFrom = [ packages.default ];
      };
    }
  );
}
```
