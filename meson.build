project(
  'GNOME Terminal', 'c',
  version: '3.25.2',
  license: 'GPL3+',
  default_options: [
    'buildtype=debugoptimized',
    'warning_level=1'
  ],
  meson_version: '>= 0.41.0'
)

gt_name = 'gnome-terminal'

gt_version = meson.project_version()
version_array = gt_version.split('.')
gt_major_version = version_array[0].to_int()
gt_minor_version = version_array[1].to_int()
gt_micro_version = version_array[2].to_int()
if version_array.length() > 3
  gt_extra_version = version_array[3].to_int()
endif

gt_prefix = get_option('prefix')
gt_bindir = join_paths(gt_prefix, get_option('bindir'))
gt_datadir = join_paths(gt_prefix, get_option('datadir'))
gt_libdir = join_paths(gt_prefix, get_option('libdir'))
gt_libexecdir = join_paths(gt_prefix, get_option('libexecdir'))
gt_localedir = join_paths(gt_prefix, get_option('localedir'))

gt_pkgdatadir = join_paths(gt_datadir, gt_name)

gt_debug = get_option('buildtype').contains('debug')

cc = meson.get_compiler('c')

config_h = configuration_data()

# Debugging
config_h.set('ENABLE_DEBUG', gt_debug)

# Distribution specifics
config_h.set('ENABLE_DISTRO_PACKAGING', get_option('enable-distro-packaging'),
             description: 'Define if building a distro package')

os_check = run_command(find_program('grep'), 'ID', '/etc/os-release')
config_h.set('HAVE_UBUNTU', os_check.stdout().contains('ubuntu'),
             description: 'Define if building under ubuntu')

# package
set_defines = [
  ['PACKAGE', gt_name],
  ['PACKAGE_BUGREPORT', 'http://bugzilla.gnome.org/enter_bug.cgi?product=' + gt_name],
  ['PACKAGE_NAME', meson.project_name()],
  ['PACKAGE_STRING', '@0@ @1@'.format(meson.project_name(), gt_version)],
  ['PACKAGE_TARNAME', gt_name],
  ['PACKAGE_URL', 'https://wiki.gnome.org/Apps/Terminal'],
  ['PACKAGE_VERSION', gt_version],
  ['VERSION', gt_version],
  # i18n
  ['GETTEXT_PACKAGE', gt_name]
]

foreach define: set_defines
  config_h.set_quoted(define[0], define[1])
endforeach

# headers
check_headers = [
  ['HAVE_DLFCN_H', 'dlfcn.h'],
  ['HAVE_INTTYPES_H', 'inttypes.h'],
  ['HAVE_LOCALE_H', 'locale.h'],
  ['HAVE_MEMORY_H', 'memory.h'],
  ['HAVE_STDINT_H', 'stdint.h'],
  ['HAVE_STDLIB_H', 'stdlib.h'],
  ['HAVE_STRINGS_H', 'strings.h'],
  ['HAVE_STRING_H', 'string.h'],
  ['HAVE_SYS_STAT_H', 'sys/stat.h'],
  ['HAVE_SYS_TYPES_H', 'sys/types.h'],
  ['HAVE_UNISTD_H', 'unistd.h']
]

foreach header: check_headers
  config_h.set(header[0], cc.has_header(header[1]))
endforeach

# functions
check_functions = [
  # i18n
  ['HAVE_DCGETTEXT', 'dcgettext'],
  ['HAVE_GETTEXT', 'gettext']
]

if host_machine.system().contains('darwin')
  check_functions += [
    ['HAVE_CFLOCALECOPYCURRENT', 'CFLocaleCopyCurrent'],
    ['HAVE_CFPREFERENCESCOPYAPPVALUE', 'CFPreferencesCopyAppValue']
  ]
endif

foreach func: check_functions
  config_h.set(func[0], cc.has_function(func[1]))
endforeach

# symbols
check_symbols = [
  ['HAVE_BIND_TEXTDOMAIN_CODESET', 'libintl.h', 'bind_textdomain_codeset'],
  ['HAVE_LC_MESSAGES', 'locale.h', 'LC_MESSAGES']
]

foreach symbol: check_symbols
  config_h.set(symbol[0], cc.has_header_symbol(symbol[1], symbol[2]))
endforeach

# compiler flags
common_flags = ['-DHAVE_CONFIG_H']
warn_flags = []

if gt_debug
  test_cflags = [
    '-fdiagnostics-show-option',
    '-fno-common',
    '-fno-strict-aliasing',
    '-fstack-protector',
    '-fstack-protector-strong',
    '-fvisibility=hidden',
    '-pipe',
    '-Waggregate-return',
    '-Wcast-align',
    '-Wendif-labels',
    '-Werror=declaration-after-statement',
    '-Werror=format=2',
    '-Werror=implicit-function-declaration',
    '-Werror=init-self',
    '-Werror=missing-include-dirs',
    '-Werror=missing-prototypes',
    '-Werror=pointer-arith',
    '-Wfloat-equal',
    '-Wformat-signedness',
    '-Wlogical-op',
    '-Wmissing-declarations',
    '-Wmissing-include-dirs',
    '-Wmissing-format-attribute',
    '-Wmissing-noreturn',
    '-Wnested-externs',
    '-Wno-missing-field-initializers',
    '-Wno-switch-enum',
    '-Wno-unused-parameter',
    '-Wold-style-definition',
    '-Wpacked',
    '-Wshadow',
    '-Wstrict-aliasing=2',
    '-Wstrict-prototypes',
    '-Wundef',
    '-Wunsafe-loop-optimizations',
    '-Wwrite-strings'
  ]

  foreach cflag: test_cflags
    if cc.has_argument(cflag)
      warn_flags += [cflag]
    endif
  endforeach
endif

add_project_arguments(common_flags, language: 'c')

platform_deps = []

gdk_dep = dependency('gdk-3.0', required: false)
targets = gdk_dep.get_pkgconfig_variable('targets')
if targets.contains('x11')
  platform_deps += dependency('x11')
endif

if targets.contains('win32') or targets.contains('quartz')
  message('unsupported GDK backend')
endif

gio_req_version = '>= 2.34.0'

vte_dep = dependency('vte-2.91', version: '>= 0.49.2')
glib_dep = dependency('glib-2.0', version: '>= 2.42.0')
gio_dep = dependency('gio-2.0', version: gio_req_version)
gio_unix_dep = dependency('gio-unix-2.0', version: gio_req_version)
gtk_dep = dependency('gtk+-3.0', version: '>= 3.12.0')
dconf_dep = dependency('dconf', version: '>= 0.14.0')
uuid_dep = dependency('uuid')

# DBus
dbus_interface_dir = get_option('with-dbus-interface-dir').strip()
if dbus_interface_dir == ''
  dbus_interface_dir = join_paths(gt_datadir, 'dbus-1', 'interfaces')
endif

dbus_service_dir = get_option('with-dbus-service-dir').strip()
if dbus_service_dir == ''
  dbus_service_dir = join_paths(gt_datadir, 'dbus-1', 'services')
endif

# GNOME Shell search provider
enable_search_provider = get_option('enable-search-provider')
if enable_search_provider
  search_provider_iface = join_paths(dbus_interface_dir, 'org.gnome.ShellSearchProvider2.xml')
  res = run_command(find_program('test'), '-e', search_provider_iface)
  assert(res.returncode() == 0, 'gnome-shell search provider requested but interface definition file not found')
endif

config_h.set('ENABLE_SEARCH_PROVIDER', enable_search_provider,
             description: 'Define to enable gnome-shell search provider')

# Vala
enable_gterminal = get_option('enable-gterminal')
if enable_gterminal
  vala_req_version = '>= 0.26'
  error_str = 'vala compiler ' + vala_req_version + ' is required for gterminal support. Try -Denable-gterminal=false'

  assert(add_languages('vala', required: false), error_str)
  valac_version = meson.get_compiler('vala').version()
  assert(valac_version.version_compare(vala_req_version), error_str)

  have_vala_0_31 = valac_version.version_compare('>= 0.31.1')
endif

# GConf-GSettings Migration
enable_migration = get_option('enable-migration')
config_h.set('ENABLE_MIGRATION', enable_migration,
             description: 'Define to 1 to enable prefs migration from GConf to GSettings')

# Nautilus extension
nautilus_dir = get_option('with-nautilus-dir').strip()

enable_nautilus_extension = get_option('enable-nautilus-extension')
if enable_nautilus_extension and nautilus_dir == ''
  nautilus_dir = join_paths(gt_libdir, 'nautilus', 'extensions-3.0')
endif

configure_file(
  output: 'config.h',
  configuration: config_h
)

gnome = import('gnome')
i18n = import('i18n')
pkg = import('pkgconfig')

top_inc = include_directories('.')

po_dir = join_paths(meson.source_root(), 'po')

intltool_merge = find_program('intltool-merge')
intltool_cache = join_paths(po_dir, '.intltool-merge-cache')
intltool_desktop_cmd = [intltool_merge, '-d', '-u', '-c', intltool_cache, po_dir, '@INPUT@', '@OUTPUT@']
intltool_xml_cmd = [intltool_merge, '-x', '-u', '-c', intltool_cache, po_dir, '@INPUT@', '@OUTPUT@']

desktop_conf = configuration_data()
desktop_conf.set('VERSION', gt_version)

desktop = 'org.gnome.Terminal.desktop'

desktop_in = configure_file(
  input: desktop + '.in.in',
  output: desktop + '.in',
  configuration: desktop_conf
)

custom_target(
  desktop,
  input: desktop_in,
  output: desktop,
  command: intltool_desktop_cmd,
  install: true,
  install_dir: join_paths(gt_datadir, 'applications')
)

appdata_xml = 'org.gnome.Terminal.appdata.xml'

custom_target(
  appdata_xml,
  input: appdata_xml + '.in',
  output: appdata_xml,
  command: intltool_xml_cmd,
  install: true,
  install_dir: join_paths(gt_datadir, 'metainfo')
)

nautilus_xml = 'org.gnome.Terminal.Nautilus.metainfo.xml'

custom_target(
  nautilus_xml,
  input: nautilus_xml + '.in',
  output: nautilus_xml,
  command: intltool_xml_cmd,
  install: true,
  install_dir: join_paths(gt_datadir, 'metainfo')
)

if get_option('enable-schemas-compile')
  meson.add_install_script('meson_post_install.py')
endif

subdir('src')
subdir('po')
subdir('help')

output = '\ngnome-terminal-' + gt_version + ':\n\n'
output += '       prefix:                 ' + gt_prefix + '\n'
output += '       source code location:   ' + meson.source_root() + '\n'
output += '       compiler:               ' + cc.get_id() + '\n'
output += '       gterminal:              ' + enable_gterminal.to_string() + '\n'
output += '       DBus interface dir:     ' + dbus_interface_dir + '\n'
output += '       DBus service dir:       ' + dbus_service_dir + '\n'
output += '       Debug:                  ' + gt_debug.to_string() + '\n'
output += '       Prefs migration:        ' + enable_migration.to_string() + '\n'
output += '       Search provider:        ' + enable_search_provider.to_string() + '\n'
output += '       Nautilus extension:     ' + enable_nautilus_extension.to_string() + '\n'
output += '       Nautilus extension dir: ' + nautilus_dir
message(output)
