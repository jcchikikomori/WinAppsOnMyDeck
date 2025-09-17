# WinAppsOnMyDeck

Documentation is purely based on the [official one](https://github.com/winapps-org/winapps/blob/main/docs/docker.md).

## Creating a Windows VM in `Docker` or `Podman`

Although WinApps supports using `QEMU+KVM+libvirt` as a backend for running Windows virtual machines, it is recommended to use `Docker` or `Podman`. These backends automate the setup process, eliminating the need for manual configuration and optimisation of the Windows virtual machine.

> [!IMPORTANT]
> Running a Windows virtual machine using `Docker` or `Podman` as a backend is only possible on GNU/Linux systems. This is due to the necessity of kernel interfaces, such as the KVM hypervisor, for achieving acceptable performance. The performance of the virtual machine can vary based on the version of the Linux kernel, with newer releases generally offering better performance.
> [!IMPORTANT]
> WinApps does __NOT__ officially support versions of Windows prior to Windows 10. Despite this, it may be possible to achieve a successful installation with some additional experimentation. If you find a way to achieve this, please share your solution through a pull request for the benefit of other users.
> Possible setup instructions for Windows 10:
>
> - 'Professional', 'Enterprise' or 'Server' editions of Windows are required to run RDP applications. Windows 'Home' will __NOT__ suffice.
> - It is recommended to edit the initial `compose.yaml` file to keep your required username and password from the beginning.
> - It is recommended to not use `sudo` to force commands to run. Add your user to the relevant permissions group wherever possible.
> [!IMPORTANT]
> The iptables kernel module must be loaded for folder sharing with the host to work.
> Check that the output of `lsmod | grep ip_tables` and `lsmod | grep iptable_nat` is non-empty.
> If the output of one of the previous commands is empty, run `echo -e "ip_tables\niptable_nat" | sudo tee /etc/modules-load.d/iptables.conf` and reboot.

### `Docker`

### Installation

You can find a guide for installing `Docker Engine` [here](https://docs.docker.com/engine/install/).

#### Setup `Docker` Container

WinApps utilises `docker compose` to configure Windows VMs. A template [`compose.yaml`](../compose.yaml) is provided.

Prior to installing Windows, you can modify the RAM and number of CPU cores available to the Windows VM by changing `RAM_SIZE` and `CPU_CORES` within `compose.yaml`.

It is also possible to specify the version of Windows you wish to install within `compose.yaml` by modifying `VERSION`.

Please refer to the [original GitHub repository](https://github.com/dockur/windows) for more information on additional configuration options.

> [!NOTE]
> If you want to undo all your changes and start from scratch, run the following. For `podman`, replace `docker compose` with `podman-compose`.
>
> ```bash
> docker compose down --rmi=all --volumes
> ```

#### Installing Windows

You can initiate the Windows installation using `docker compose`.

```bash
cd winapps
docker compose --file ./compose.yaml up
```

You can then access the Windows virtual machine via a VNC connection to complete the Windows setup by navigating to <http://127.0.0.1:8006> in your web browser.

#### Changing `compose.yaml`

Changes to `compose.yaml` require the container to be removed and re-created. This should __NOT__ affect your data.

```bash
# Stop and remove the existing container.
docker compose --file ~/.config/winapps/compose.yaml down

# Remove the existing FreeRDP certificate (if required).
# Note: A new certificate will be created when connecting via RDP for the first time.
rm ~/.config/freerdp/server/127.0.0.1_3389.pem

# Re-create the container with the updated configuration.
# Add the -d flag at the end to run the container in the background.
docker compose --file ~/.config/winapps/compose.yaml up
```

#### Subsequent Use

```bash
docker compose --file ~/.config/winapps/compose.yaml start # Power on the Windows VM
docker compose --file ~/.config/winapps/compose.yaml pause # Pause the Windows VM
docker compose --file ~/.config/winapps/compose.yaml unpause # Resume the Windows VM
docker compose --file ~/.config/winapps/compose.yaml restart # Restart the Windows VM
docker compose --file ~/.config/winapps/compose.yaml stop # Gracefully shut down the Windows VM
docker compose --file ~/.config/winapps/compose.yaml kill # Force shut down the Windows VM
```

## Steam Deck Notes

- Recommended backend: Prefer `Docker` (rootless/systemless) or `libvirt` on SteamOS. Traditional rootful Docker installs and mutable system changes can be fragile on Deck; rootless/systemless Docker or `libvirt` are more resilient across updates.
- FreeRDP requirement: Install FreeRDP v3 so the `xfreerdp3` client is available on the system. WinApps uses FreeRDP v3; older clients are not supported.
- RDP security flag: Ensure `RDP_FLAGS` includes `/sec:tls` to allow connecting to the VM using TLS with the default self-signed certificate. This reduces strict certificate validation and is less secure; consider replacing the certificate with a trusted one in production setups.
- Command selection: If auto-detection fails, explicitly set `FREERDP_COMMAND` to `xfreerdp3` in your config.

Example `~/.config/winapps/winapps.conf` entries:

```ini
# Use TLS and accept self-signed cert on first use
RDP_FLAGS="/sec:tls /cert:tofu /sound /microphone +home-drive"

# Force FreeRDP v3 client
FREERDP_COMMAND="xfreerdp3"
```
