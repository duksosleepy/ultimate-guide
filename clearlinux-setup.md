# Table of contents
1. [Requirements](#setup)
2. [Available Packages](#packages)
3. [Extra packages (manual installation)](#extras)
4. [WireGuard configuration](#wire)
5. [Using Clear Linux as a KVM Host](#kvm)
# Requirements<a name="setup"></a>

### Check kernel
```bash
    uname -a
```

### The Clear Linux Installer partition names convention
* CLR_BOOT
* CLR_SWAP
* CLR_ROOT
* CLR_HOME
* CLR_MNT_/home

### The base install is really small which is great in terms of disk space used. However, it’s missing a lot of common tools. A couple things you may want to consider running:
```bash
    sudo swupd autoupdate --disable
    sudo swupd update
    sudo swupd diagnose
    sudo swupd repair
    flatpak update
    sudo swupd mirror --max-parallel-downloads=20
    sudo swupd bundle-add wget network-basic clr-network-troubleshooter
    sudo swupd bundle-add sysadmin-basic sudo (will give you basic things like “top”).
    sudo swupd search someprogram (will search for someprogram and tell you what bundle it’s found in).
    mkdir ~/code
    mkdir ~/.bashrc.d
    chmod 700 ~/.bashrc.d
    mkdir -p ~/.local/share/gnome-shell/extensions
    sudo mkdir -p /usr/local/bin
    sudo mkdir -p /usr/local/lib
    sudo mkdir -p /opt
    sudo systemctl start sshd
    sudo systemctl enable sshd
```

```bash
    for file in ~/.bashrc.d/*.bashrc;
    do
        source "$file"
    done
```

```bash
    chmod +x ~/.bashrc.d/*.bashrc
```
### Note that Clear Linux takes a very minimalistic approach, as evidenced by the very tiny amount of stuff in /etc . This means that if you say… install nginx (swupd bundle-add web-server-basic)… nginx won’t actually run until you create an /etc/nginx/nginx.conf file.


### Increase limit for inotify watches
```bash
    sudo mkdir -p /etc/sysctl.d
    cat /proc/sys/fs/inotify/max_user_watches
    echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
    sudo sysctl -p
    sudo sysctl --system
```

### Enable udev rules
```bash
    sudo mkdir -p /etc/udev/rules.d/
    # move any*.rules file into the /etc/udev/rules.d folder
    # and manually reloaded udev rules
    sudo udevadm control --reload-rules && sudo udevadm trigger
```

### Add ~/.local/bin to your PATH
```bash
    # ~/.bashrc
    export PATH=$PATH:~/.local/bin
```

### Very good solution if you use slow HDD or SSD, also can save some wear on SSD
```bash
    cp -r /var/lib/swupd /tmp/swupd
    swupd update -S /tmp/swupd && swupd 3rd-party update -S /tmp/swupd
    rm -rf /var/lib/swupd
    mv /tmp/swupd /var/lib/swupd
```

### Post install
```bash
    # delete orca autostart
    rm -rf /usr/share/gdm/greeter/autostart/orca.desktop

    # useless if you're not an Evolution user
    rm -rf /usr/share/xdg/autostart/org.gnome.Evolution-alarm-notify.desktop

    # if you don't use gnome-flackback
    rm -rf /usr/share/xdg/autostart/gnome-flackback*

    # don't needed if flathub already added
    rm -rf /usr/share/xdg/autostart/org.clearlinux.initFlathubRepo.desktop
```

### Service tweaks
```bash
    # disable GNOME Software autostart
    echo '#' > /usr/share/dbus-1/services/org.gnome.Software.service

    # disable coredump service
    ln -sf /dev/null /usr/lib/sysctl.d/50-coredump.conf


    systemctl mask cupsd.service
    systemctl mask mcelog.service
    systemctl mask pacdiscovery.service
    systemctl mask pacrunner.service
    systemctl mask swupd-overdue.service
    systemctl mask wpa_supplicant.service
    systemctl mask cupsd.socket
    systemctl mask pcscd.socket
    systemctl mask motd-update.path
    systemctl mask pacdiscovery.path
    systemctl mask cupsd.service
    systemctl mask mcelog.service
    systemctl mask ModemManager.service
    systemctl mask swap.target
    systemctl mask packagekit.service
    systemctl mask packagekit-offline-update.service

    # must-have for SSD
    systemctl enable fstrim.timer

    # disable journal storage
    vi /usr/lib/systemd/journald.conf.d/*.conf
    Storage=volatile
```

### Fonts load fix for many apps
```bash
    f=/etc/environment; s='export FONTCONFIG_PATH=/usr/share/defaults/fonts'; touch $f; if ! grep -q "$s" $f; then echo $s >> $f; fi

    # Wayland
    echo 'FONTCONFIG_PATH=/usr/share/defaults/fonts' > ~/.config/environment.d/envvars.conf

    # X11
    echo 'export FONTCONFIG_PATH=/usr/share/defaults/fonts' > ~/.xinitrc

```

### Flatpak dark theme fix
```bash
    Add Exec=env GTK_THEME=Adwaita:dark ... to every .desktop in ~/.local/share/flatpak/exports/share.
    #Add Nautilus 'New File' menu

    touch ~/Templates/"Untitled Document"
    #All systemd config paths

    /etc/systemd/system.conf,
    /etc/systemd/system.conf.d/*.conf,
    /run/systemd/system.conf.d/*.conf,
    /usr/lib/systemd/system.conf.d/*.conf
```

### Powersave
Clear performance mode is totally overkill for laptops, this guide designed to improve the situation and get better time usage with battery.
Disable Clear Linux OS enforcement of certain power and performance settings: `sudo systemctl mask clr-power.timer`
Disable turbo boost: `echo 1 | sudo tee /sys/devices/system/cpu/intel_pstate/no_turbo`

`vim /etc/systemd/system/powersave.service`

```ini
    [Unit]
    Description=Set CPU performance governor

    [Service]
    Type=oneshot
    RemainAfterExit=yes
    ExecStart=/bin/bash -c "echo active > /sys/devices/system/cpu/intel_pstate/status; echo powersave | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor"
    ExecStop=/bin/bash -c "echo performance | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor"

    [Install]
    WantedBy=multi-user.target
```

```bash
    systemctl start powersave && systemctl enable powersave
```

```bash
    mkdir -p /etc/kernel/cmdline.d
    echo "pcie_aspm.policy=powersupersave" | tee /etc/kernel/cmdline.d/aspm.conf
    clr-boot-manager update
```

### Powertop service:
```bash
    wget https://src.fedoraproject.org/rpms/powertop/raw/main/f/powertop.service
    mv powertop.service /etc/systemd/system
    systemctl daemon-reload
    systemctl enable powertop
```

### Network manager
* "Allow password only for this user" to increase security - NM store passwords in plain configs by default, not the best decison
* Add `autoconnect-priority=1` `autoconnect-retries=10` to prefered connection config for better stability
* Set `dhcp-send-hostname=false` for both `[ipv4]` & `[ipv6]` - great for privacy
* Enable `iwd` backend and systemd-resolved
```ini
    [device]
    wifi.backend=iwd

    [main]
    dns=systemd-resolved
```

### DoT setup
`vi /etc/systemd/resolved.conf.d/dns_over_tls.conf`
```ini
    DNSOverTLS=yes
    DNS=9.9.9.9#dns.quad9.net
```

Avoid Google fallback DNS
`vi /etc/systemd/resolved.conf.d/fallback_dns.conf`
```ini
    [Resolve]
    FallbackDNS=1.1.1.1 1.0.0.1 2606:4700:4700::1111 2606:4700:4700::1001

    # to disable fallback completely
    FallbackDNS=
```
`systemctl restart systemd-resolved.service`

### macOS-like fonts
Based on https://aswinmohan.me/posts/better-fonts-on-linux/
```xml
    # vim /etc/fonts/local.conf

    <?xml version='1.0'?>
    <!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
    <fontconfig>

    <match target="font">
    <edit name="autohint" mode="assign">
        <bool>true</bool>
    </edit>
    <edit name="hinting" mode="assign">
        <bool>true</bool>
    </edit>
    <edit mode="assign" name="hintstyle">
        <const>hintslight</const>
    </edit>
    <edit mode="assign" name="lcdfilter">
    <const>lcddefault</const>
    </edit>
    </match>


    <!-- Default sans-serif font -->
    <match target="pattern">
    <test qual="any" name="family"><string>-apple-system</string></test>
    <!--<test qual="any" name="lang"><string>ja</string></test>-->
    <edit name="family" mode="prepend" binding="same"><string>Tex Gyre Heros</string>  </edit>
    </match>

    <match target="pattern">
    <test qual="any" name="family"><string>Helvetica Neue</string></test>
    <!--<test qual="any" name="lang"><string>ja</string></test>-->
    <edit name="family" mode="prepend" binding="same"><string>Tex Gyre Heros</string>  </edit>
    </match>

    <match target="pattern">
    <test qual="any" name="family"><string>Helvetica</string></test>
    <!--<test qual="any" name="lang"><string>ja</string></test>-->
    <edit name="family" mode="prepend" binding="same"><string>Tex Gyre Heros</string>  </edit>
    </match>

    <match target="pattern">
    <test qual="any" name="family"><string>arial</string></test>
    <!--<test qual="any" name="lang"><string>ja</string></test>-->
    <edit name="family" mode="prepend" binding="same"><string>Tex Gyre Heros</string>  </edit>
    </match>

    <match target="pattern">
    <test qual="any" name="family"><string>sans-serif</string></test>
    <!--<test qual="any" name="lang"><string>ja</string></test>-->
    <edit name="family" mode="prepend" binding="same"><string>Tex Gyre Heros</string>  </edit>
    </match>

    <!-- Default serif fonts -->
    <match target="pattern">
    <test qual="any" name="family"><string>serif</string></test>
    <edit name="family" mode="prepend" binding="same"><string>Libertinus Serif</string>  </edit>
    <edit name="family" mode="prepend" binding="same"><string>Noto Serif</string>  </edit>
    <edit name="family" mode="prepend" binding="same"><string>Noto Color Emoji</string>  </edit>
    <edit name="family" mode="append" binding="same"><string>IPAPMincho</string>  </edit>
    <edit name="family" mode="append" binding="same"><string>HanaMinA</string>  </edit>
    </match>

    <!-- Default monospace fonts -->
    <match target="pattern">
    <test qual="any" name="family"><string>SFMono-Regular</string></test>
    <edit name="family" mode="prepend" binding="same"><string>DM Mono</string></edit>
    <edit name="family" mode="prepend" binding="same"><string>Space Mono</string></edit>
    <edit name="family" mode="append" binding="same"><string>Inconsolatazi4</string></edit>
    <edit name="family" mode="append" binding="same"><string>IPAGothic</string></edit>
    </match>

    <match target="pattern">
    <test qual="any" name="family"><string>Menlo</string></test>
    <edit name="family" mode="prepend" binding="same"><string>DM Mono</string></edit>
    <edit name="family" mode="prepend" binding="same"><string>Space Mono</string></edit>
    <edit name="family" mode="append" binding="same"><string>Inconsolatazi4</string></edit>
    <edit name="family" mode="append" binding="same"><string>IPAGothic</string></edit>
    </match>

    <match target="pattern">
    <test qual="any" name="family"><string>monospace</string></test>
    <edit name="family" mode="prepend" binding="same"><string>DM Mono</string></edit>
    <edit name="family" mode="prepend" binding="same"><string>Space Mono</string></edit>
    <edit name="family" mode="append" binding="same"><string>Inconsolatazi4</string></edit>
    <edit name="family" mode="append" binding="same"><string>IPAGothic</string></edit>
    </match>

    <!-- Fallback fonts preference order -->
    <alias>
    <family>sans-serif</family>
    <prefer>
    <family>Noto Sans</family>
    <family>Noto Color Emoji</family>
    <family>Noto Emoji</family>
    <family>Open Sans</family>
    <family>Droid Sans</family>
    <family>Ubuntu</family>
    <family>Roboto</family>
    <family>NotoSansCJK</family>
    <family>Source Han Sans JP</family>
    <family>IPAPGothic</family>
    <family>VL PGothic</family>
    <family>Koruri</family>
    </prefer>
    </alias>
    <alias>
    <family>serif</family>
    <prefer>
    <family>Noto Serif</family>
    <family>Noto Color Emoji</family>
    <family>Noto Emoji</family>
    <family>Droid Serif</family>
    <family>Roboto Slab</family>
    <family>IPAPMincho</family>
    </prefer>
    </alias>
    <alias>
    <family>monospace</family>
    <prefer>
    <family>Noto Sans Mono</family>
    <family>Noto Color Emoji</family>
    <family>Noto Emoji</family>
    <family>Inconsolatazi4</family>
    <family>Ubuntu Mono</family>
    <family>Droid Sans Mono</family>
    <family>Roboto Mono</family>
    <family>IPAGothic</family>
    </prefer>
    </alias>

    </fontconfig>
```
Browser -> Settings -> Select Customize Fonts under Appearences -> Choose SF fonts

# Available Packages<a name="packages"></a>


# Extra packages (manual installation)<a name="extras"></a>

### Gnome Extensions
```bash
    mv ~/Downloads/clipboard-indicator@tudmotu.com.v26.shell-extension ~/.local/share/gnome-shell/extensions/clipboard-indicator@tudmotu.com
```
```bash
    mv ~/Downloads/clipboard-indicator@tudmotu.com.v26.shell-extension  ~/.local/share/gnome-shell/extensions/caffeine@patapon.info
```

### H264 in Firefox
```bash
    #!/usr/bin/env bash

    ## References:
    ## https://community.clearlinux.org/t/how-to-h264-etc-support-for-firefox-including-ffmpeg-install

    ## Install dependencies
    echo -e "\e[33m\xe2\x8f\xb3 Install the following dependencies: 'c-basic', 'devpkg-libva', and 'git' ...\e[m"
    sudo swupd bundle-add c-basic devpkg-libva git

    ## Clone the ffmpeg git repository if it doesn't exist; or update it if it exists
    echo -e "\e[33m\xe2\x8f\xb3 Get latest FFmpeg source repository ...\e[m"
    if [ -d "FFmpeg" ];then
    # shellcheck disable=SC2164
    cd FFmpeg
    git fetch
    else
    git clone https://github.com/FFmpeg/FFmpeg
    # shellcheck disable=SC2164
    cd FFmpeg
    fi

    ## Get the latest non-dev release
    echo -e "\e[33m\xe2\x8f\xb3 Checkout the latest non-dev release ...\e[m"
    git checkout tags/"$(git tag -l | sed -n -E '/^n[[:digit:]]+\.[[:digit:]]+\.[[:digit:]]+$/p' | sort | tail -1)"

    ## Build FFmpeg, which would be installed under /opt/ffmpeg
    echo -e "\e[33m\xe2\x8f\xb3 Building ...\e[m"
    echo -e "\e[32m If the installation is successful, it would be available under \e[33m/opt/ffmpeg\e[32m.\e[m"
    read -rp "Press any key to continue ... " -n1 -s
    echo
    if ! ./configure --prefix=/opt/ffmpeg --enable-shared || ! make || ! sudo make install; then
    echo -e "\e[31m Installation failed! Aborting...\e[m"
    exit 1
    fi

    ## Configure the dynamic linker configuration to include /opt/ffmpeg/lib
    echo -e "\e[33m\xe2\x8f\xb3 Configuring dynamic linker configuration ...\e[m"
    if [ ! -f /etc/ld.so.conf ] || \
    grep -q 'include /etc/ld\.so\.conf\.d/\*\.conf' /etc/ld.so.conf; then
    printf "include /etc/ld.so.conf.d/*.conf" | sudo tee -a /etc/ld.so.conf
    fi
    if [ ! -d /etc/ld.so.conf.d ]; then
    sudo mkdir /etc/ld.so.conf.d
    fi
    if [ ! -f /etc/ld.so.conf.d/ffmpeg.conf ] || \
    ! grep -q '/opt/ffmpeg/lib' /etc/ld.so.conf.d/ffmpeg.conf; then
    echo "/opt/ffmpeg/lib" | sudo tee /etc/ld.so.conf.d/ffmpeg.conf
    fi
    echo -e "\e[32m Updating dynamic linker run-time bindings and library cache ...\e[m"
    sudo ldconfig

    ## Add ffmpeg to library path of Firefox
    echo -e "\e[33m\xe2\x8f\xb3 Add FFmpeg to libarry path of Firefox ...\e[m"
    if [ ! -f "${HOME}/.config/firefox.conf" ] || \
    grep -q 'export LD_LIBRARY_PATH' "${HOME}/.config/firefox.conf"; then
    echo "export LD_LIBRARY_PATH=/opt/ffmpeg/lib" >> "${HOME}/.config/firefox.conf"
    else
    grep -q 'export LD_LIBRARY_PATH=.*/opt/ffmpeg/lib.*' "${HOME}/.config/firefox.conf" \
        || sed -i 's#export LD_LIBRARY_PATH=#&/opt/ffmpeg/lib:#' "${HOME}/.config/firefox.conf"
    fi
```

### DBeaver
Download latest tar.gz archiv of community edition for Linux 64bit from dbeaver.com and extract to /opt/dbeaver.
Add icon to GNOME like so: `gnome-desktop-item-edit ~/.local/share/applications/ --create-new`

```ini
    […]
    Icon[en_US]=dbeaver
    Exec=/opt/dbeaver/dbeaver
    Name[en_US]=DBeaver
    Name=DBeaver
    Icon=dbeaver
```

# WireGuard configuration
Based on [WireGuard configuration](https://bacnh.com/how-to-install-wireguard-in-clearlinux)


### To let peer access the network of a VPN server - meaning connect to Internet or to access all my local HomeLab IP, need to enable IP fowarding.
```bash
    sysctl -w net.ipv4.ip_forward=1
    sysctl -w net.ipv6.conf.all.forwarding=1
```

### Setup traffic rules to accept all UDP traffic via port 51820 and also accept conntrack traffics
```bash
    #IPTables track connections
    iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

    #Setup Wireguard
    iptables -A INPUT -p udp -m udp --dport 51820 -m conntrack --ctstate NEW -j ACCEPT
```
or

```bash
    # Save firewall config
    sudo systemctl start iptables-save

    # Enable iptables-restore, so config can be loaded at booting time
    sudo systemctl enable iptables-restore.service
```

### Create a Wireguard configuration. Just keep it for now, as we can just use the config generator or you can follow WireGuard quick start to create config manually.
`nano /etc/wireguard/wg0.conf`
```ini
    [Interface]
    PrivateKey = 8GboYh0YF3q/hJhoPFoL3HM/ObgOuC8YI6UXWsgWL2M=
    Address = 10.10.10.2/32
    DNS = 192.168.1.254

    [Peer]
    PublicKey = OwdegSTyhlpw7Dbpg8VSUBKXF9CxoQp2gAOdwgqtPVI=
    AllowedIPs = 0.0.0.0/0
    Endpoint = vpn.example.com:51820
```

or

```ini
    [Interface]
    Address = 10.10.10.1/24
    DNS = 192.168.1.254
    ListenPort = 51820
    PrivateKey = YNqHwpcAmVj0lVzPSt3oUnL7cRPKB/geVxccs0C0kk0=
    [Peer]
    PublicKey = CLnGaiAfyf6kTBJKh0M529MnlqfFqoWJ5K4IAJ2+X08=
    AllowedIPs = 10.10.10.2/32
```

### Start wg-quick@wg0service to create wireguard network and listens to the peer:
```bash
    sudo systemctl enable wg-quick@wg0.service
    sudo systemctl start wg-quick@wg0.service
```

### Last step, I need to open port 51820 that point to the ClearLinux KVM (192.168.11.20). In my router running OpenWrt, this can be done [link](https://bacnh.com/how-to-build-openwrt-image-for-your-router-using-archlinux-container):
```ini
config redirect
	option dest_port '51820'
	option src 'wan'
	option name 'WireGuard-VPN'
	option src_dport '51820'
	option target 'DNAT'
	option dest_ip '192.168.11.20'
	option dest 'lan'
	list proto 'udp'
```
### Check the network, clients info
```bash
    sudo wg
```

# RequireUsing Clear Linux as a KVM Host<a name="kvm"></a>
Based on https://marks.page/blog/clear_kvm/ and https://brooks.sh/2017/12/22/configuring-kvm-on-clear-linux/


### Add the following bundles:
```bash
    sudo swupd bundle-add kernel-kvm kvm-host
```

### The first installs virtualization features in the kernel, and the second installs packages that that allow you to manage a kvm instance.
To create virtual machines, you must either login as root or add your user to the kvm user group:
```bash
    sudo usermod -aG kvm [username]
```

### Finally, we will utilize libvirtd to manage the virtual machines. To enable this toolkit’s service:
```bash
    sudo systemctl enable libvirtd
```

### list currently configured interfaces
```bash
    nmcli device status
```
### Take note of the interface that you currently use for internet access. Now lets create a new bridge called br0:
```bash
    sudo nmcli con add type bridge ifname br0 con-name br0
```

### We will link it to the eno1 adapter for physical access:
```bash
    sudo nmcli con add type bridge-slave ifname eno1 master br0
```

### (Optional) Disabled STP for the bridge (we generally don’t need our bridge part of the network tree):
```bash
    sudo nmcli con modify br0 bridge.stp no
```

### By default, NetworkManager assumes you will use DHCP. However, to set a static IP (replacing the corresponding fields for your network):
```bash
    sudo nmcli con modify br0 ipv4.method manual ipv4.address "[addr/prefix_bit_length]" ipv4.gateway [gateway]" ipv4.dns 1.1.1.1 ipv4.dns-search "example.tld"
```

### Now it’s time to bring down the the eno1 interface connection:
```bash
    sudo nmcli con down eno1
```

### And bring up the bridge:
```bash
    sudo nmcli con up br0
```

### prevent packets forwarded through the bridge from routing to the iptables of the host. We can append the following 3 lines to a new file for our bridge in /etc/sysctl.d/ directory:
```bash
    sudo nano /etc/sysctl.d/0-bridge.conf
```
```conf
    net.bridge.bridge-nf-call-arptables = 0
    net.bridge.bridge-nf-call-ip6tables = 0
    net.bridge.bridge-nf-call-iptables = 0
```

### Finally, a restart of the system:
```bash
    sudo reboot
```
