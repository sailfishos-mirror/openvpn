Client Options
--------------
The client options are used when connecting to an OpenVPN server configured
to use ``--server``, ``--server-bridge``, or ``--mode server`` in its
configuration.

--allow-pull-fqdn
  Allow client to pull DNS names from server (rather than being limited to
  IP address) for ``--ifconfig``, ``--route``, and ``--route-gateway``.

--allow-recursive-routing
  When this option is set, OpenVPN will not drop incoming tun packets with
  same destination as host.

--auth-token token
  This is not an option to be used directly in any configuration files,
  but rather push this option from a ``--client-connect`` script or a
  ``--plugin`` which hooks into the :code:`OPENVPN_PLUGIN_CLIENT_CONNECT`
  or :code:`OPENVPN_PLUGIN_CLIENT_CONNECT_V2` calls. This option provides a
  possibility to replace the clients password with an authentication token
  during the lifetime of the OpenVPN client.

  Whenever the connection is renegotiated and the
  ``--auth-user-pass-verify`` script or ``--plugin`` making use of the
  :code:`OPENVPN_PLUGIN_AUTH_USER_PASS_VERIFY` hook is triggered, it will
  pass over this token as the password instead of the password the user
  provided. The authentication token can only be reset by a full reconnect
  where the server can push new options to the client. The password the
  user entered is never preserved once an authentication token has been
  set. If the OpenVPN server side rejects the authentication token then
  the client will receive an :code:`AUTH_FAILED` and disconnect.

  The purpose of this is to enable two factor authentication methods, such
  as HOTP or TOTP, to be used without needing to retrieve a new OTP code
  each time the connection is renegotiated. Another use case is to cache
  authentication data on the client without needing to have the users
  password cached in memory during the life time of the session.

  To make use of this feature, the ``--client-connect`` script or
  ``--plugin`` needs to put
  ::

     push "auth-token UNIQUE_TOKEN_VALUE"

  into the file/buffer for dynamic configuration data. This will then make
  the OpenVPN server to push this value to the client, which replaces the
  local password with the ``UNIQUE_TOKEN_VALUE``.

  Newer clients (2.4.7+) will fall back to the original password method
  after a failed auth. Older clients will keep using the token value and
  react according to ``--auth-retry``

--auth-token-user base64username
  Companion option to ``--auth-token``. This options allows one to override
  the username used by the client when reauthenticating with the ``auth-token``.
  It also allows one to use ``--auth-token`` in setups that normally do not use
  username and password.

  The username has to be base64 encoded.

--auth-user-pass
  Authenticate with server using username/password.

  Valid syntaxes:
  ::

      auth-user-pass
      auth-user-pass up

  If ``up`` is present, it must be a file containing username/password on 2
  lines. If the password line is missing, OpenVPN will prompt for one.

  If ``up`` is omitted, username/password will be prompted from the
  console.

  This option can also be inlined
  ::

    <auth-user-pass>
    username
    [password]
    </auth-user-pass>

  where password is optional, and will be prompted from the console if
  missing.

  The server configuration must specify an ``--auth-user-pass-verify``
  script to verify the username/password provided by the client.

--auth-retry type
  Controls how OpenVPN responds to username/password verification errors
  such as the client-side response to an :code:`AUTH_FAILED` message from
  the server or verification failure of the private key password.

  Normally used to prevent auth errors from being fatal on the client
  side, and to permit username/password requeries in case of error.

  An :code:`AUTH_FAILED` message is generated by the server if the client
  fails ``--auth-user-pass`` authentication, or if the server-side
  ``--client-connect`` script returns an error status when the client
  tries to connect.

  ``type`` can be one of:

  :code:`none`
      Client will exit with a fatal error (this is the default).

  :code:`nointeract`
      Client will retry the connection without requerying
      for an ``--auth-user-pass`` username/password. Use this option for
      unattended clients.

  :code:`interact`
      Client will requery for an ``--auth-user-pass``
      username/password and/or private key password before attempting a
      reconnection.

  Note that while this option cannot be pushed, it can be controlled from
  the management interface.

--client
  A helper directive designed to simplify the configuration of OpenVPN's
  client mode. This directive is equivalent to:
  ::

       pull
       tls-client

--client-nat args
  This pushable client option sets up a stateless one-to-one NAT rule on
  packet addresses (not ports), and is useful in cases where routes or
  ifconfig settings pushed to the client would create an IP numbering
  conflict.

  Examples:
  ::

      client-nat snat 192.168.0.0/255.255.0.0
      client-nat dnat 10.64.0.0/255.255.0.0

  ``network/netmask`` (for example :code:`192.168.0.0/255.255.0.0`) defines
  the local view of a resource from the client perspective, while
  ``alias/netmask`` (for example :code:`10.64.0.0/255.255.0.0`) defines the
  remote view from the server perspective.

  Use :code:`snat` (source NAT) for resources owned by the client and
  :code:`dnat` (destination NAT) for remote resources.

  Set ``--verb 6`` for debugging info showing the transformation of
  src/dest addresses in packets.

--connect-retry args
  Wait ``n`` seconds between connection attempts (default :code:`1`).
  Repeated reconnection attempts are slowed down after 5 retries per
  remote by doubling the wait time after each unsuccessful attempt.

  Valid syntaxes:
  ::

     connect retry n
     connect retry n max

  If the optional argument ``max`` is specified, the maximum wait time in
  seconds gets capped at that value (default :code:`300`).

--connect-retry-max n
  ``n`` specifies the number of times each ``--remote`` or
  ``<connection>`` entry is tried. Specifying ``n`` as :code:`1` would try
  each entry exactly once. A successful connection resets the counter.
  (default *unlimited*).

--connect-timeout n
  See ``--server-poll-timeout``.

--dns args
  Client DNS configuration to be used with the connection.

  Valid syntaxes:
  ::

     dns search-domains domain [domain ...]
     dns server n address addr[:port] [addr[:port] ...]
     dns server n resolve-domains domain [domain ...]
     dns server n dnssec yes|optional|no
     dns server n transport DoH|DoT|plain
     dns server n sni server-name

  The ``--dns search-domains`` directive takes one or more domain names
  to be added as DNS domain suffixes. If it is repeated multiple times within
  a configuration the domains are appended, thus e.g. domain names pushed by
  a server will amend locally defined ones.

  The ``--dns server`` directive is used to configure DNS server ``n``.
  The server id ``n`` must be a value between -128 and 127. For pushed
  DNS server options it must be between 0 and 127. The server id is used
  to group options and also for ordering the list of configured DNS servers;
  lower numbers come first. DNS servers being pushed to a client replace
  already configured DNS servers with the same server id.

  The ``address`` option configures the IPv4 and / or IPv6 address(es) of
  the DNS server. Up to eight addresses can be specified per DNS server.
  Optionally a port can be appended after a colon. IPv6 addresses need to
  be enclosed in brackets if a port is appended.

  The ``resolve-domains`` option takes one or more DNS domains used to define
  a split-dns or dns-routing setup, where only the given domains are resolved
  by the server. Systems which do not support fine grained DNS domain
  configuration will ignore this setting.

  The ``dnssec`` option is used to configure validation of DNSSEC records.
  While the exact semantics may differ for resolvers on different systems,
  ``yes`` likely makes validation mandatory, ``no`` disables it, and ``optional``
  uses it opportunistically.

  The ``transport`` option enables DNS-over-HTTPS (``DoH``) or DNS-over-TLS (``DoT``)
  for a DNS server. The ``sni`` option can be used with them to specify the
  ``server-name`` for TLS server name indication.

  Each server has to have at least one address configured for a configuration
  to be valid. All the other options can be omitted.

  Note that not all options may be supported on all platforms. As soon support
  for different systems is implemented, information will be added here how
  unsupported options are treated.

  The ``--dns`` option will eventually obsolete the ``--dhcp-option`` directive.
  Until then it will replace configuration at the places ``--dhcp-option`` puts it,
  so that ``--dns`` overrides ``--dhcp-option``. Thus, ``--dns`` can be used today
  to migrate from ``--dhcp-option``.

--explicit-exit-notify n
  In UDP client mode or point-to-point mode, send server/peer an exit
  notification if tunnel is restarted or OpenVPN process is exited. In
  client mode, on exit/restart, this option will tell the server to
  immediately close its client instance object rather than waiting for a
  timeout.

  If both server and client support sending this message using the control
  channel, the message will be sent as control-channel message. Otherwise
  the message is sent as data-channel message, which will be ignored by
  data-channel offloaded peers.

  The **n** parameter (default :code:`1` if not present) controls the
  maximum number of attempts that the client will try to resend the exit
  notification message if messages are sent in data-channel mode.

  In UDP server mode, send :code:`RESTART` control channel command to
  connected clients. The ``n`` parameter (default :code:`1` if not present)
  controls client behavior. With ``n`` = :code:`1` client will attempt to
  reconnect to the same server, with ``n`` = :code:`2` client will advance
  to the next server.

  OpenVPN will not send any exit notifications unless this option is
  enabled.

--inactive args
  Causes OpenVPN to exit after ``n`` seconds of inactivity on the TUN/TAP
  device. The time length of inactivity is measured since the last
  incoming or outgoing tunnel packet. The default value is 0 seconds,
  which disables this feature.

  Valid syntaxes:
  ::

       inactive n
       inactive n bytes

  If the optional ``bytes`` parameter is included, exit if less than
  ``bytes`` of combined in/out traffic are produced on the tun/tap device
  in ``n`` seconds.

  In any case, OpenVPN's internal ping packets (which are just keepalives)
  and TLS control packets are not considered "activity", nor are they
  counted as traffic, as they are used internally by OpenVPN and are not
  an indication of actual user activity.

--proto-force p
  When iterating through connection profiles, only consider profiles using
  protocol ``p`` (:code:`tcp` \| :code:`udp`).

  Note that this specifically only filters by the transport layer
  protocol, i.e. UDP or TCP. This does not affect whether IPv4 or
  IPv6 is used as IP protocol.

  For implementation reasons the option accepts the :code:`4` and :code:`6`
  suffixes when specifying the protocol
  (i.e. :code:`udp4` / :code:`udp6` / :code:`tcp4` / :code:`tcp6`).
  However, these behave the same as without the suffix and should be avoided
  to prevent confusion.

--pull
  This option must be used on a client which is connecting to a
  multi-client server. It indicates to OpenVPN that it should accept
  options pushed by the server, provided they are part of the legal set of
  pushable options (note that the ``--pull`` option is implied by
  ``--client`` ).

  In particular, ``--pull`` allows the server to push routes to the
  client, so you should not use ``--pull`` or ``--client`` in situations
  where you don't trust the server to have control over the client's
  routing table.

--pull-filter args
  Filter options on the client pushed by the server to the client.

  Valid syntaxes:
  ::

     pull-filter accept text
     pull-filter ignore text
     pull-filter reject text

  Filter options received from the server if the option starts with
  :code:`text`.  The action flag :code:`accept` allows the option,
  :code:`ignore` removes it and :code:`reject` flags an error and triggers
  a :code:`SIGUSR1` restart. The filters may be specified multiple times,
  and each filter is applied in the order it is specified. The filtering of
  each option stops as soon as a match is found. Unmatched options are accepted
  by default.

  Prefix comparison is used to match :code:`text` against the received option so
  that
  ::

      pull-filter ignore "route"

  would remove all pushed options starting with ``route`` which would
  include, for example, ``route-gateway``. Enclose *text* in quotes to
  embed spaces.

  ::

      pull-filter accept "route 192.168.1."
      pull-filter ignore "route "

  would remove all routes that do not start with ``192.168.1``.

  *Note* that :code:`reject` may result in a repeated cycle of failure and
  reconnect, unless multiple remotes are specified and connection to the
  next remote succeeds. To silently ignore an option pushed by the server,
  use :code:`ignore`.

--push-peer-info
  Push additional information about the client to server. The following
  data is always pushed to the server:

  :code:`IV_VER=<version>`
        The client OpenVPN version

  :code:`IV_PLAT=[linux|solaris|openbsd|mac|netbsd|freebsd|win]`
        The client OS platform

  :code:`IV_PROTO`
    Details about protocol extensions that the peer supports. The
    variable is a bitfield and the bits are defined as follows:

    - bit 0: Reserved, should always be zero
    - bit 1: The peer supports peer-id floating mechanism
    - bit 2: The client expects a push-reply and the server may
      send this reply without waiting for a push-request first.
    - bit 3: The client is capable of doing key derivation using
      RFC5705 key material exporter.
    - bit 4: The client is capable of accepting additional arguments
      to the ``AUTH_PENDING`` message.
    - bit 5: The client supports doing feature negotiation in P2P mode
    - bit 6: The client is capable of parsing and receiving the ``--dns`` pushed option
    - bit 7: The client is capable of sending exit notification via control channel using ``EXIT`` message. Also, the client is accepting the protocol-flags pushed option for the EKM capability
    - bit 8: The client is capable of accepting ``AUTH_FAILED,TEMP`` messages
    - bit 9: The client is capable of dynamic tls-crypt
    - bit 10: The client is capable of data epoch keys

  :code:`IV_NCP=2`
        Negotiable ciphers, client supports ``--cipher`` pushed by
        the server, a value of 2 or greater indicates client supports
        *AES-GCM-128* and *AES-GCM-256*. IV_NCP is *deprecated* in
        favor of ``IV_CIPHERS``.

  :code:`IV_CIPHERS=<data-ciphers>`
        The client announces the list of supported ciphers configured with the
        ``--data-ciphers`` option to the server.

  :code:`IV_MTU=<max_mtu>`
        The client announces the support of pushable MTU and the maximum MTU
        it is willing to accept.

  :code:`IV_GUI_VER=<gui_id> <version>`
        The UI version of a UI if one is running, for example
        :code:`de.blinkt.openvpn 0.5.47` for the Android app.
        This may be set by the client UI/GUI using ``--setenv``.

  :code:`IV_SSO=[crtext,][openurl,][proxy_url]`
        Additional authentication methods supported by the client.
        This may be set by the client UI/GUI using ``--setenv``.

  The following flags depend on which compression formats are compiled in
  and whether compression is allowed by options. See `Protocol options`_
  for more details.

    :code:`IV_LZO=1`
        If client supports LZO compression.

    :code:`IV_LZO_STUB=1`
        If client was built with LZO stub capability. This is only sent if
        ``IV_LZO=1`` is not sent. This means the client can talk to a server
        configured with ``--comp-lzo no``.

    :code:`IV_LZ4=1` and :code:`IV_LZ4v2=1`
        If the client supports LZ4 compression.

    :code:`IV_COMP_STUB=1` and :code:`IV_COMP_STUBv2=1`
        If the client supports stub compression. This means the client can talk
        to a server configured with ``--compress``.

  When ``--push-peer-info`` is enabled the additional information consists
  of the following data:

  :code:`IV_HWADDR=<string>`
        This is intended to be a unique and persistent ID of the client.
        The string value can be any readable ASCII string up to 64 bytes.
        OpenVPN 2.x and some other implementations use the MAC address of
        the client's interface used to reach the default gateway. If this
        string is generated by the client, it should be consistent and
        preserved across independent sessions and preferably
        re-installations and upgrades.

  :code:`IV_SSL=<version string>`
        The ssl library version used by the client, e.g.
        :code:`OpenSSL 1.0.2f 28 Jan 2016`.

  :code:`IV_PLAT_VER=x.y`
        The version of the operating system, e.g. 6.1 for Windows 7.
        This may be set by the client UI/GUI using ``--setenv``.
        On Windows systems it is automatically determined by openvpn
        itself.  On other platforms OpenVPN will default to sending
        the information returned by the `uname()` system call in
        the `release` field, which is usually the currently running
        kernel version.  This is highly system specific, though.

  :code:`UV_<name>=<value>`
        Client environment variables whose names start with
        :code:`UV_`

--remote args
  Remote host name or IP address, port and protocol.

  Valid syntaxes:
  ::

     remote host
     remote host port
     remote host port proto

  The ``port`` and ``proto`` arguments are optional. The OpenVPN client
  will try to connect to a server at ``host:port``.  The ``proto`` argument
  indicates the protocol to use when connecting with the remote, and may be
  :code:`tcp` or :code:`udp`.  To enforce IPv4 or IPv6 connections add a
  :code:`4` or :code:`6` suffix; like :code:`udp4` / :code:`udp6`
  / :code:`tcp4` / :code:`tcp6`.

  On the client, multiple ``--remote`` options may be specified for
  redundancy, each referring to a different OpenVPN server, in the order
  specified by the list of ``--remote`` options. Specifying multiple
  ``--remote`` options for this purpose is a special case of the more
  general connection-profile feature. See the ``<connection>``
  documentation below.

  The client will move on to the next host in the list, in the event of
  connection failure. Note that at any given time, the OpenVPN client will
  at most be connected to one server.

  Examples:
  ::

     remote server1.example.net
     remote server1.example.net 1194
     remote server2.example.net 1194 tcp

  *Note:*
     Since UDP is connectionless, connection failure is defined by
     the ``--ping`` and ``--ping-restart`` options.

     Also, if you use multiple ``--remote`` options, AND you are dropping
     root privileges on the client with ``--user`` and/or ``--group`` AND
     the client is running a non-Windows OS, if the client needs to switch
     to a different server, and that server pushes back different TUN/TAP
     or route settings, the client may lack the necessary privileges to
     close and reopen the TUN/TAP interface. This could cause the client
     to exit with a fatal error.

  If ``--remote`` is unspecified, OpenVPN will listen for packets from any
  IP address, but will not act on those packets unless they pass all
  authentication tests. This requirement for authentication is binding on
  all potential peers, even those from known and supposedly trusted IP
  addresses (it is very easy to forge a source IP address on a UDP
  packet).

  When used in TCP mode, ``--remote`` will act as a filter, rejecting
  connections from any host which does not match ``host``.

  If ``host`` is a DNS name which resolves to multiple IP addresses,
  OpenVPN will try them in the order that the system getaddrinfo()
  presents them, so priorization and DNS randomization is done by the
  system library. Unless an IP version is forced by the protocol
  specification (4/6 suffix), OpenVPN will try both IPv4 and IPv6
  addresses, in the order getaddrinfo() returns them.

--remote-random
  When multiple ``--remote`` address/ports are specified, or if connection
  profiles are being used, initially randomize the order of the list as a
  kind of basic load-balancing measure.

--remote-random-hostname
  Prepend a random string (6 bytes, 12 hex characters) to hostname to
  prevent DNS caching. For example, "foo.bar.gov" would be modified to
  "<random-chars>.foo.bar.gov".

--resolv-retry n
  If hostname resolve fails for ``--remote``, retry resolve for ``n``
  seconds before failing.

  Set ``n`` to :code:`infinite` to retry indefinitely.

  By default, ``--resolv-retry infinite`` is enabled. You can disable by
  setting n=0.

--single-session
  After initially connecting to a remote peer, disallow any new
  connections. Using this option means that a remote peer cannot connect,
  disconnect, and then reconnect.

  If the daemon is reset by a signal or ``--ping-restart``, it will allow
  one new connection.

  ``--single-session`` can be used with ``--ping-exit`` or ``--inactive``
  to create a single dynamic session that will exit when finished.

--server-poll-timeout n
  When connecting to a remote server do not wait for more than ``n``
  seconds for a response before trying the next server. The default value
  is :code:`120`. This timeout includes proxy and TCP connect timeouts.

--static-challenge args
  Enable static challenge/response protocol

  Valid syntax:
  ::

     static-challenge text echo [format]

  The ``text`` challenge text is presented to the user which describes what
  information is requested.  The ``echo`` flag indicates if the user's
  input should be echoed on the screen.  Valid ``echo`` values are
  :code:`0` or :code:`1`. The optional ``format`` indicates whether
  the password and response should be combined using the SCRV1 protocol
  (``format`` = :code:`scrv1`) or simply concatenated (``format`` = :code:`concat`).
  :code:`scrv1` is the default.

  See management-notes.txt in the OpenVPN distribution for a description of
  the OpenVPN challenge/response protocol.

.. include:: proxy-options.rst
