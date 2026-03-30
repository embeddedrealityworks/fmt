project('fmt', 'cpp', 'c',
  version: '12.1.1',
  license: 'MIT',
  default_options: [
    'cpp_std=c++11',
    'warning_level=1',
  ],
  meson_version: '>=1.1.0',
)

cpp = meson.get_compiler('cpp')

fmt_os      = get_option('fmt_os')
fmt_unicode = get_option('fmt_unicode')
fmt_pedantic = get_option('fmt_pedantic')
fmt_werror  = get_option('fmt_werror')
fmt_install = get_option('fmt_install')
fmt_tests   = get_option('fmt_tests')

inc = include_directories('include')

# Per-compiler pedantic flags
pedantic_flags = []
if fmt_pedantic
  if cpp.get_id() == 'gcc'
    pedantic_flags = [
      '-pedantic-errors', '-Wall', '-Wextra', '-pedantic',
      '-Wold-style-cast', '-Wundef', '-Wredundant-decls',
      '-Wwrite-strings', '-Wpointer-arith', '-Wcast-qual',
      '-Wformat=2', '-Wmissing-include-dirs', '-Wcast-align',
      '-Wctor-dtor-privacy', '-Wdisabled-optimization', '-Winvalid-pch',
      '-Woverloaded-virtual', '-Wconversion', '-Wno-ctor-dtor-privacy',
      '-Wno-format-nonliteral',
    ]
  elif cpp.get_id() == 'clang'
    pedantic_flags = [
      '-Wall', '-Wextra', '-pedantic', '-Wconversion', '-Wundef',
      '-Wdeprecated', '-Wweak-vtables', '-Wshadow',
      '-Wno-gnu-zero-variadic-macro-arguments',
    ]
    if cpp.has_argument('-Wzero-as-null-pointer-constant')
      pedantic_flags += '-Wzero-as-null-pointer-constant'
    endif
  elif cpp.get_id() == 'msvc'
    pedantic_flags = ['/W3']
  endif
endif

werror_flag = []
if fmt_werror
  if cpp.get_id() in ['gcc', 'clang']
    werror_flag = ['-Werror']
  elif cpp.get_id() == 'msvc'
    werror_flag = ['/WX']
  endif
endif

# Unicode on MSVC requires /utf-8
utf8_flag = []
if host_machine.system() == 'windows' and cpp.get_id() == 'msvc'
  if fmt_unicode
    utf8_flag = ['/utf-8']
  endif
endif

fmt_compile_defs = []
if not fmt_unicode and host_machine.system() == 'windows' and cpp.get_id() == 'msvc'
  fmt_compile_defs += '-DFMT_UNICODE=0'
endif

# fmt library
fmt_sources = ['src/format.cc']
if fmt_os
  fmt_sources += 'src/os.cc'
else
  fmt_compile_defs += '-DFMT_OS=0'
endif

fmt_lib = library('fmt',
  sources: fmt_sources,
  include_directories: inc,
  cpp_args: fmt_compile_defs + pedantic_flags + werror_flag + utf8_flag,
  install: fmt_install,
  version: meson.project_version(),
  soversion: '12',
)

fmt_dep = declare_dependency(
  link_with: fmt_lib,
  include_directories: inc,
  compile_args: fmt_compile_defs,
)

# fmt-header-only (interface / header-only variant)
fmt_header_only_dep = declare_dependency(
  include_directories: inc,
  compile_args: ['-DFMT_HEADER_ONLY=1'],
)

# fmt-c library (C API wrapper)
fmt_c_extra = []
if host_machine.system() == 'windows' and cpp.get_id() == 'msvc'
  fmt_c_extra = ['/Zc:preprocessor']
endif

fmt_c_lib = static_library('fmt-c',
  sources: 'src/fmt-c.cc',
  include_directories: inc,
  cpp_args: fmt_c_extra,
  link_with: fmt_lib,
  install: fmt_install,
)

fmt_c_dep = declare_dependency(
  link_with: [fmt_c_lib, fmt_lib],
  include_directories: inc,
)

# Install
if fmt_install
  fmt_headers = [
    'include/fmt/args.h',
    'include/fmt/base.h',
    'include/fmt/chrono.h',
    'include/fmt/color.h',
    'include/fmt/compile.h',
    'include/fmt/core.h',
    'include/fmt/fmt-c.h',
    'include/fmt/format.h',
    'include/fmt/format-inl.h',
    'include/fmt/os.h',
    'include/fmt/ostream.h',
    'include/fmt/printf.h',
    'include/fmt/ranges.h',
    'include/fmt/std.h',
    'include/fmt/xchar.h',
  ]
  install_headers(fmt_headers, subdir: 'fmt')

  pkgconfig = import('pkgconfig')
  pkgconfig.generate(fmt_lib,
    name: 'fmt',
    description: 'A modern formatting library',
    url: 'https://fmt.dev',
    filebase: 'fmt',
  )
endif

# Tests
if fmt_tests
  subdir('test')
endif
