## Get Started with NixOS


After learning the basics of the Nix language, we can start using it to configure our NixOS. The default configuration for NixOS is located at `/etc/nixos/configuration.nix`, which contains all the declarative configuration for the system, such as time zone, language, keyboard layout, network, users, file system, boot options, etc.

If we want to modify the system state in a reproducible way (**which is also the most recommended way**), we need to manually edit `/etc/nixos/configuration.nix`, and then execute `sudo nixos-rebuild switch` to apply the modified configuration, it will generate a new system environment based on the configuration file we modified, sets the new environment as the default one, and also preserves & added the previous environment into the boot options of grub/sytemd-boot. This ensures we can always roll back to the old environment(even if the new environment fails to start).

`/etc/nixos/configuration.nix` is the classic method to configure NixOS, which relies on data sources configured by `nix-channel` and has no version-locking mechanism, making it difficult to ensure the reproducibility of the system. **A better approach is to use Flakes**, which can ensure the reproducibility of the system and make it easy to manage the configuration.

Now first, let's learn how to manage NixOS through the classic method, `/etc/nixos/configuration.nix`, and then migrate to the more advanced Flakes.

## Configuring the system using `/etc/nixos/configuration.nix`

As I mentioned earlier, this is the classic method to configured NixOS, and also the default method currently used by NixOS. It relies on data sources configured by `nix-channel` and has no version-locking mechanism, making it difficult to ensure the reproducibility of the system.

For example, to enable ssh and add a user "ryan", simply add the following content into `/etc/nixos/configuration.nix`:

```nix
# Edit this configuration file to define what should be installed on
# your system.  Help is available in the configuration.nix(5) man page
# and in the NixOS manual (accessible by running 'nixos-help').
{ config, pkgs, ... }:

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];

  # Omit the previous configuration.......

  # add user ryan
  users.users.ryan = {
    isNormalUser = true;
    description = "ryan";
    extraGroups = [ "networkmanager" "wheel" ];
    openssh.authorizedKeys.keys = [
        # replace with your own public key
        "ssh-ed25519 <some-public-key> ryan@ryan-pc"
    ];
    packages = with pkgs; [
      firefox
    #  thunderbird
    ];
  };

  # enable openssh-server
  services.openssh = {
    enable = true;
    permitRootLogin = "no";         # disable root login
    passwordAuthentication = false; # disable password login
    openFirewall = true;
    forwardX11 = true;              # enable X11 forwarding
  };

  # omit the rest of the configuration.......
}
```

In this configuration, we declared that we want to enable the openssh service, add an ssh public key for the user ryan, and disable password login.

Now, let's run `sudo nixos-rebuild switch` to deploy the modified configuration, and then we can login to the system using ssh with the ssh keys we configured.

Any reproducible changes to the system can be made by modifying `/etc/nixos/configuration.nix` and deploying the changes by running `sudo nixos-rebuild switch`.

All configuration options in `/etc/nixos/configuration.nix` can be found in the following places:

- By searching on Google, such as `Chrome NixOS`, which will provide NixOS informations related to Chrome. Generally, the NixOS Wiki and the source code of Nixpkgs will be among the top results.
- By searching for keywords in [NixOS Options Search](https://search.nixos.org/options).
- For system-level configuration, relevant documentation can be found in [Configuration - NixOS Manual](https://nixos.org/manual/nixos/unstable/index.html#ch-configuration).
- By searching for keywords directly in the source code of [nixpkgs](https://github.com/NixOS/nixpkgs) on GitHub.


## References

- [Overview of the NixOS Linux distribution]( https://nixos.wiki/wiki/Overview_of_the_NixOS_Linux_distribution)
