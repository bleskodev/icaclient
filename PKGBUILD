# Maintainer: Stephan Springer <buzo+arch@Lini.de>
# Contributor: Eric Liu <eric@hnws.me>
# Contributor: Daniele Vazzola <daniele.vazzola@gmail.com>
# Contributor: Ciarán Coffey <ciaran@ccoffey.ie>
# Contributor: Matthew Gyurgyik <matthew@pyther.net>
# Contributor: Giorgio Azzinnaro <giorgio@azzinna.ro>

pkgname=icaclient
pkgver=22.9.0.21
pkgrel=1
pkgdesc="Citrix Workspace App (a.k.a. ICAClient, Citrix Receiver)"
arch=('x86_64' 'i686' 'armv7h')
url='https://www.citrix.com/downloads/workspace-app/linux/workspace-app-for-linux-latest.html'
license=('custom:Citrix')
depends=('alsa-lib' 'curl' 'gst-plugins-base-libs' 'gtk2' 'libc++' 'libc++abi' 'libidn11'
         'libjpeg6-turbo' 'libpng12' 'libsecret' 'libsoup' 'libvorbis' 'libxaw' 'libxp'
         'openssl' 'speex' 'webkit2gtk')
optdepends=('xerces-c: gtk2 configuration manager'
            'webkit2gtk: gtk2 selfservice/storefront ui'
            'libc++: for HDXTeams')
conflicts=('bin32-citrix-client' 'citrix-client')
options=(!strip)
backup=("opt/Citrix/ICAClient/config/appsrv.ini" "opt/Citrix/ICAClient/config/wfclient.ini" "opt/Citrix/ICAClient/config/module.ini")
_dl_urls_="$(curl -sL "$url" | grep -F ".tar.gz?__gda__")"
_dl_urls="$(echo "$_dl_urls_" | grep -F "$pkgver.tar.gz?__gda__")"
_source32=https:"$(echo "$_dl_urls" | sed -En 's|^.*rel="(//.*/linuxx86-[^"]*)".*$|\1|p')"
_source64=https:"$(echo "$_dl_urls" | sed -En 's|^.*rel="(//.*/linuxx64-[^"]*)".*$|\1|p')"
_sourcearmhf=https:"$(echo "$_dl_urls" | sed -En 's|^.*rel="(//.*/linuxarmhf-[^"]*)".*$|\1|p')"
source=('citrix-configmgr.desktop'
        'citrix-conncenter.desktop'
        'citrix-wfica.desktop'
        'citrix-workspace.desktop'
        'ctxusbd.service'
        'wfica.sh'
        'wfica_assoc.sh')
source_x86_64=("$pkgname-x64-$pkgver.tar.gz::$_source64")
source_i686=("$pkgname-x86-$pkgver.tar.gz::$_source32")
source_armv7h=("$pkgname-armhf-$pkgver.tar.gz::$_sourcearmhf")
sha256sums=('643427b6e04fc47cd7d514af2c2349948d3b45f536c434ba8682dcb1d4314736'
            '446bfe50e5e1cb027415b264a090cede1468dfbdc8b55e5ce14e9289b6134119'
            '1dc6d6592fa08c44fb6a4efa0dc238e9e78352bb799ef2e1a92358b390868064'
            'cdfb3a2ef3bf6b0dd9d17c7a279735db23bc54420f34bfd43606830557a922fe'
            '775bcf855af4be15783126cffe6ae0900da5e1554bcbed6cd94e93856460bd04'
            'fe0b92bb9bfa32010fe304da5427d9ca106e968bad0e62a5a569e3323a57443f'
            'a3bd74aaf19123cc550cde71b5870d7dacf9883b7e7a85c90e03b508426c16c4')
sha256sums_x86_64=('34ddb5874816e509fe006186e2b40245b59e509db1c020d92908c2155065a2e6')
sha256sums_i686=('94c318f2a0e36407222f072cd59eb34900c525d99631139cc9bf1fdfa896ee05')
sha256sums_armv7h=('9e77162eb6505b849b5894d02b1685d80a7ab02ce3a2515c682f6d61b9ded80b')
install=citrix-client.install

installLog() {
    cd ${srcdir}

    local LogFilesLocation=$1

    local ServicePath="${pkgdir}/usr/lib/systemd/system"
    local ServiceName="ctxlogd.service"
    mkdir -p "${ServicePath}"

    sed -e "s,###ICAROOT###,$ICAROOT,g" \
        -e "s,###USER###,multi-user,g" \
        -e "s,###CitrixUser###,User=citrixlog,g" \
        < "${LogFilesLocation}/${ServiceName}" > "${ServicePath}/${ServiceName}"
}

installUSB() {
    cd ${srcdir}

    local UsbFilesLocation=$1

    # copy the files
    install -m555  -t "${pkgdir}$ICAROOT" "${UsbFilesLocation}/VDGUSB.DLL"
    install -m544  -t "${pkgdir}$ICAROOT" "${UsbFilesLocation}/ctxusbd"
    install -m4555 -t "${pkgdir}$ICAROOT" "${UsbFilesLocation}/ctxusb"
    install -m555  -t "${pkgdir}$ICAROOT" "${UsbFilesLocation}/ctx_usb_isactive"
    install -m644  -t "${pkgdir}$ICAROOT" "${UsbFilesLocation}/usb.conf"

    # install the udev rule
    local UdevRulesDir="${pkgdir}/etc/udev/rules.d"
    mkdir -p ${UdevRulesDir}
    sed -e "s,###ICAROOT###,$ICAROOT,g" \
        < "${UsbFilesLocation}/ica-usb.rules" > "$UdevRulesDir/85-ica-usb.rules"
    chmod 444 "$UdevRulesDir/85-ica-usb.rules"

    # install daemon script
    sed -e "s,###ICAROOT###,$ICAROOT,g" \
        -e "s,###INIT_UDEV###,udev,g" \
        < "${UsbFilesLocation}/ctxusbd.rc" > "${pkgdir}$ICAROOT/ctxusbd.rc"
    chmod 544 "${pkgdir}$ICAROOT/ctxusbd.rc"

    # install systemd service files
    install -m644 -D -t "${pkgdir}/usr/lib/systemd/system" ctxusbd.service

    # patch module.ini to include Generic USB
    local ModuleIniDir=${pkgdir}$ICAROOT/config

    sed -e 's/^[ \t]*VirtualDriver[ \t]*=.*$/&, GenericUSB/' \
        -e '/\[ICA 3.0\]/a\GenericUSB=on' \
         < "${ModuleIniDir}/module.ini" > "${ModuleIniDir}/new_module.ini"
    echo -e "\n[GenericUSB]" >> "${ModuleIniDir}/new_module.ini"
    echo DriverName = VDGUSB.DLL >> "${ModuleIniDir}/new_module.ini"
    mv "${ModuleIniDir}/new_module.ini" "${ModuleIniDir}/module.ini"

    # patch usb.conf file to add missing fallback rule (might not be needed in the future)
    echo -e "\nCONNECT: # Ultimate fallback: allow anything else" >> "${pkgdir}$ICAROOT/usb.conf"
}

package() {
    cd "${srcdir}"
    ICAROOT=/opt/Citrix/ICAClient
    if [[ $CARCH == 'i686' ]]
    then
        ICADIR="$srcdir/linuxx86/linuxx86.cor"
        PKGINF="Ver.core.linuxx86"
    elif [[ $CARCH == 'x86_64' ]]
    then
        ICADIR="$srcdir/linuxx64/linuxx64.cor"
        PKGINF="Ver.core.linuxx64"
    elif [[ $CARCH == 'armv7h' ]]
    then
        ICADIR="$srcdir/linuxarmhf/linuxarmhf.cor"
        PKGINF="Ver.core.linuxarmhf"
    fi

    mkdir -p "${pkgdir}$ICAROOT"

    cd "$ICADIR"
    install -m755 -t "${pkgdir}$ICAROOT" \
            *.so *.DLL \
            adapter AuthManagerDaemon icasessionmgr NativeMessagingHost \
            PrimaryAuthManager ServiceRecord selfservice UtilDaemon wfica

    # copy directories
    cp -rt "${pkgdir}$ICAROOT" config gtk help icons keyboard keystore lib nls site util
    # fix permissions
    chmod -R a+r "${pkgdir}$ICAROOT"

    rm "${pkgdir}$ICAROOT/lib/UIDialogLibWebKit.so"

    # Install License
    install -m644 -D -t "${pkgdir}$ICAROOT" nls/en.UTF-8/eula.txt

    # Install Version
    install -m644 -D "${srcdir}/PkgId" "${pkgdir}$ICAROOT/pkginf/$PKGINF"

    # create /config/.server to enable user customization using ~/.ICACLient/ overrides. Thanks Tomek
    touch "${pkgdir}$ICAROOT/config/.server"

    # Install wrapper script
    install -m755 "${srcdir}/wfica.sh" "${pkgdir}$ICAROOT/wfica.sh"

    ln -s gst_play1.0 "${pkgdir}/$ICAROOT/util/gst_play"
    ln -s gst_read1.0 "${pkgdir}/$ICAROOT/util/gst_read"

    # Dirty Hack
    # wfica expects {module,wfclient,apssrv}.ini in $ICAROOT/config
    # sadly these configs differ slightly by locale
    lang=${LANG%%_*}
    [[ -d "${pkgdir}/$ICAROOT/nls/$lang" ]] || lang='en'
    cp "${pkgdir}$ICAROOT/nls/$lang/module.ini" "${pkgdir}/$ICAROOT/config/"
    cp "${pkgdir}$ICAROOT/nls/$lang/appsrv.template" "${pkgdir}/$ICAROOT/config/appsrv.ini"
    cp "${pkgdir}$ICAROOT/nls/$lang/wfclient.template" "${pkgdir}/$ICAROOT/config/wfclient.ini"

    sed -i \
        -e 's/Ceip=Enable/Ceip=Disable/' \
        -e 's/DisableHeartBeat=False/DisableHeartBeat=True/' \
        "${pkgdir}$ICAROOT/config/module.ini"
    cd "${srcdir}"
    # install freedesktop.org files
    install -Dm644 -t "$pkgdir"/usr/share/applications citrix-{configmgr,conncenter,workspace,wfica}.desktop
    # install scripts
    install -Dm755 -t "${pkgdir}$ICAROOT" wfica.sh wfica_assoc.sh
    chmod +x "${pkgdir}$ICAROOT"/util/{HdxRtcEngine,ctx_app_bind,ctxlogd,icalicense.sh,setlog}

    # make certificates available
    rm -r "${pkgdir}/opt/Citrix/ICAClient/keystore/cacerts"
    ln -s /etc/ssl/certs "${pkgdir}/opt/Citrix/ICAClient/keystore/cacerts"

    # Log Daemon needs special install procedure
    installLog $ICADIR

    # USB requres special install procedure
    installUSB $ICADIR/usb
}
