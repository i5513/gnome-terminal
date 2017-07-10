service_conf = configuration_data()
service_conf.set('libexecdir', gt_libexecdir)

service = 'org.gnome.Terminal.service'

configure_file(
  input: service + '.in',
  output: service,
  install: true,
  install_dir: dbus_service_dir,
  configuration: service_conf
)

service = 'gnome-terminal-server.service'

configure_file(
  input: service + '.in',
  output: service,
  install: true,
  install_dir: join_paths(gt_libdir, 'systemd', 'user'),
  configuration: service_conf
)

install_data(
  'org.gnome.Terminal.gschema.xml',
  install_dir: join_paths(gt_datadir, 'glib-2.0', 'schemas')
)

enum_headers = files('terminal-enums.h')

enum = 'terminal-type-builtins'

gt_enums = gnome.mkenums(
  enum,
  sources: enum_headers,
  c_template: enum + '.c.template',
  h_template: enum + '.h.template'
)

service = 'org.gnome.Terminal'
namespace = 'Terminal'

gt_gdbus = gnome.gdbus_codegen(
  'terminal-gdbus-generated',
  service + '.xml',
  interface_prefix: service,
  namespace: namespace,
  object_manager: true
)

gt_deps = platform_deps + [
  dconf_dep,
  gio_dep,
  glib_dep,
  gtk_dep,
  uuid_dep,
  vte_dep,
  dependency('gsettings-desktop-schemas', version: '>= 0.1.0'),
  dependency('libpcre2-8', version: '>= 10.00'),
  dependency('threads')
]

# gnome-terminal-server
sources = files(
  'eggshell.c',
  'profile-editor.c',
  'server.c',
  'terminal-accels.c',
  'terminal-app.c',
  'terminal-debug.c',
  'terminal-encoding.c',
  'terminal-gdbus.c',
  'terminal-icon-button.c',
  'terminal-info-bar.c',
  'terminal-i18n.c',
  'terminal-mdi-container.c',
  'terminal-notebook.c',
  'terminal-prefs.c',
  'terminal-profiles-list.c',
  'terminal-settings-list.c',
  'terminal-screen.c',
  'terminal-screen-container.c',
  'terminal-search-popover.c',
  'terminal-tab-label.c',
  'terminal-tabs-menu.c',
  'terminal-util.c',
  'terminal-window.c'
)

if enable_search_provider
  sources += files('terminal-search-provider.c')

  search_provider = 'terminal-search-provider-gdbus-generated'

  sources += gnome.gdbus_codegen(
    search_provider,
    search_provider_iface,
    interface_prefix: 'org.gnome.Shell',
    namespace: namespace
  )

  install_data(
    'gnome-terminal-search-provider.ini',
    install_dir: join_paths(gt_datadir, 'gnome-shell', 'search-providers')
  )
endif

marshal = 'terminal-marshal'

sources += gnome.genmarshal(
  marshal,
  sources: marshal + '.list',
  prefix: '_terminal_marshal',
  internal: true
)

resource_data = files(
  'preferences.ui',
  'profile-preferences.ui',
  'search-popover.ui',
  'terminal-menus.ui',
  'terminal-window.ui',
  'terminal.about',
  'terminal.common.css',
  'terminal.xml'
)

sources += gnome.compile_resources(
  'terminal-resources',
  'terminal.gresource.xml',
  source_dir: '.',
  c_name: 'terminal',
  dependencies: resource_data,
  export: true
)

version_conf = configuration_data()
version_conf.set('TERMINAL_MAJOR_VERSION', gt_major_version)
version_conf.set('TERMINAL_MINOR_VERSION', gt_minor_version)
version_conf.set('TERMINAL_MICRO_VERSION', gt_micro_version)

version = 'terminal-version.h'

sources += configure_file(
  input: version + '.in',
  output: version,
  configuration: version_conf
)

cflags = warn_flags + [
  '-DTERMINAL_COMPILATION',
  '-DTERM_LIBEXECDIR="@0@"'.format(gt_libexecdir),
  '-DTERM_LOCALEDIR="@0@"'.format(gt_localedir)
]

executable(
  gt_name + '-server',
  sources + [gt_enums, gt_gdbus],
  include_directories: top_inc,
  dependencies: gt_deps,
  c_args: cflags,
  install: true,
  install_dir: gt_libexecdir
)

# Legacy terminal client
sources = files(
  'terminal.c',
  'terminal-client-utils.c',
  'terminal-debug.c',
  'terminal-i18n.c',
  'terminal-options.c',
  'terminal-profiles-list.c',
  'terminal-settings-list.c'
)

cflags = warn_flags + [
  '-DTERMINAL_COMPILATION',
  '-DTERMINAL_CLIENT',
  '-DTERM_DATADIR="@0@"'.format(gt_datadir),
  '-DTERM_LOCALEDIR="@0@"'.format(gt_localedir),
  '-DTERM_PKGDATADIR="@0@"'.format(gt_pkgdatadir)
]

executable(
  gt_name,
  sources + [gt_enums, gt_gdbus],
  include_directories: top_inc,
  dependencies: gt_deps,
  c_args: cflags,
  install: true,
  install_dir: gt_bindir
)

# Checks
test_name = 'terminal-regex'

exe = executable(
  test_name,
  test_name + '.c',
  include_directories: top_inc,
  c_args: warn_flags + ['-DTERMINAL_REGEX_MAIN'],
  dependencies: gt_deps
)

test(test_name, exe)

# Terminal client
if enable_gterminal
  sources = files(
    'gterminal.vala',
    'client.vapi',
    'config.vapi',
    'profiles.vapi',
    'terminal-client-utils.c',
    'terminal-debug.c',
    'terminal-profiles-list.c',
    'terminal-settings-list.c'
  )

  deps = platform_deps + [
    dconf_dep,
    gio_dep,
    gio_unix_dep,
    glib_dep,
    gtk_dep,
    # FIXME: use of uuid dependency on vala fails due to missing vapi file
    # https://github.com/mesonbuild/meson/issues/1195
    # https://github.com/mesonbuild/meson/issues/1923
    #uuid_dep
  ]

  cflags = [
    '-DTERMINAL_COMPILATION',
    '-DTERMINAL_CLIENT',
    '-DLOCALEDIR="@0@"'.format(gt_localedir),
    '-DGETTEXT_PACKAGE="@0@"'.format(gt_name)
  ]

  # See bug #710862 about -Wsuggest-attribute=format
  test_cflags = [
    '-Wno-cast-qual',
    '-Wno-format-nonliteral',
    '-Wno-suggest-attribute=format',
    '-Wno-unused-but-set-variable',
    '-Wno-unused-function',
    '-Wno-unused-variable',
    '-Wno-write-strings'
  ]

  foreach cflag: test_cflags
    if cc.has_argument(cflag)
      cflags += [cflag]
    endif
  endforeach

  executable(
    'gterminal',
    sources + [gt_enums],
    include_directories: top_inc,
    dependencies: deps,
    # FIXME: uuid include path should be removed
    c_args: cflags + ['-I/usr/include/uuid'],
    vala_args: '--pkg=posix',
    # FIXME: uuid lib should be removed
    link_args: '-luuid'
  )
endif

# Pref migrator
if enable_migration
  sources = files(
    'migration.c',
    'terminal-debug.c',
    'terminal-profiles-list.c',
    'terminal-settings-list.c'
  )

  deps = [
    dconf_dep,
    gio_dep,
    uuid_dep,
    vte_dep,
    dependency('gconf-2.0', version: '>= 2.31.3')
  ]

  executable(
    gt_name + '-migration',
    sources + [gt_enums],
    include_directories: top_inc,
    dependencies: deps,
    c_args: warn_flags,
    install: true,
    install_dir: gt_libexecdir
  )
endif

# Nautilus extension
if enable_nautilus_extension
  sources = files(
    'terminal-client-utils.c',
    'terminal-i18n.c',
    'terminal-nautilus.c'
  )

  deps = [
    gio_dep,
    gio_unix_dep,
    dependency('libnautilus-extension', version: '>= 3.0.0')
  ]

  symbol_map = 'nautilus.map'

  ldflag = '-Wl,--version-script,@0@/@1@'.format(meson.current_source_dir(), symbol_map)

  ldflags = []
  if host_machine.system().contains('linux') and cc.has_argument(ldflag)
    ldflags += ldflag
  endif

  shared_module(
    'terminal-nautilus',
    sources: sources + [gt_enums, gt_gdbus],
    include_directories: top_inc,
    dependencies: deps,
    c_args: warn_flags + ['-DTERM_LOCALEDIR="@0@"'.format(gt_localedir)],
    link_args: ldflags,
    link_depends: symbol_map,
    install: true,
    install_dir: nautilus_dir
  )
endif
