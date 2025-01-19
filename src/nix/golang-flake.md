# Golang Flake


``` nix
{
  description = "A flake example";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };

  outputs = { self, nixpkgs, ... }:
    let
      system = "x86_64-linux";
      pkgs = nixpkgs.legacyPackages.${system};
    in
      {
      packages.${system}.default = pkgs.buildGoModule {
        pname = "package-name";
        version = "1.0.0";

        src = ./.;
        vendorHash = ""; # Leave empty to get the hash from nix run
      };

      apps.${system}.default =
        let
          # Example of installing a runtime dependency
          ld_library_path = pkgs.lib.makeLibraryPath [ pkgs.ffmpeg ];
          path = pkgs.lib.makeBinPath [ pkgs.ffmpeg ];
          prog = pkgs.writeShellScriptBin "run-package-name" ''
          export LD_LIBRARY_PATH=${ld_library_path}
          export PATH=${path}:$PATH
          exec ${self.packages.${system}.default}/bin/package-name "$@"
          '';
        in
        {
          type = "app";
          program = "${prog}/bin/run-package-name";
        };
    };
}
```
