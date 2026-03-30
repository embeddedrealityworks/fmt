test_inc = include_directories('.', '../include')

# ── Bundled GTest / GMock ─────────────────────────────────────────────────────
thread_dep = dependency('threads', required: false)

gtest_compile_args = ['-DGTEST_HAS_STD_WSTRING=1']
if not thread_dep.found()
  gtest_compile_args += '-DGTEST_HAS_PTHREAD=0'
endif

gtest_lib = static_library('gtest',
  sources: 'gtest/gmock-gtest-all.cc',
  include_directories: include_directories('gtest'),
  cpp_args: gtest_compile_args,
  dependencies: thread_dep.found() ? [thread_dep] : [],
  override_options: ['cpp_std=c++11', 'warning_level=0'],
)

gtest_dep = declare_dependency(
  link_with: gtest_lib,
  include_directories: include_directories('gtest'),
  compile_args: gtest_compile_args,
  dependencies: thread_dep.found() ? [thread_dep] : [],
)

# ── test-main (shared test harness) ──────────────────────────────────────────
test_main_lib = static_library('test-main',
  sources: ['test-main.cc', 'gtest-extra.cc', 'util.cc'],
  include_directories: test_inc,
  dependencies: [gtest_dep, fmt_dep],
  override_options: ['warning_level=0'],
)

test_main_dep = declare_dependency(
  link_with: test_main_lib,
  include_directories: test_inc,
  dependencies: [gtest_dep, fmt_dep],
)

# ── Helper ────────────────────────────────────────────────────────────────────
# Adds an executable and registers it as a Meson test.
# add_fmt_test(name, extra_sources, extra_deps, extra_args)

# Standard tests: link against test-main (which pulls in gtest + fmt)
standard_tests = [
  'args-test',
  'assert-test',
  'base-test',
  'chrono-test',
  'color-test',
  'compile-test',
  'gtest-extra-test',
  'ostream-test',
  'printf-test',
  'std-test',
  'xchar-test',
]

foreach t : standard_tests
  exe = executable(t,
    sources: t + '.cc',
    dependencies: [test_main_dep],
    override_options: ['warning_level=0'],
  )
  test(t, exe)
endforeach

# format-test (references mock-allocator.h — header only, no extra source)
format_test_exe = executable('format-test',
  sources: 'format-test.cc',
  dependencies: [test_main_dep],
  override_options: ['warning_level=0'],
)
test('format-test', format_test_exe)

# ranges-test needs ranges-odr-test.cc as a second translation unit
ranges_test_exe = executable('ranges-test',
  sources: ['ranges-test.cc', 'ranges-odr-test.cc'],
  dependencies: [test_main_dep],
  override_options: ['warning_level=0'],
)
test('ranges-test', ranges_test_exe)

# enforce-checks-test requires FMT_ENFORCE_COMPILE_STRING
enforce_checks_exe = executable('enforce-checks-test',
  sources: 'enforce-checks-test.cc',
  cpp_args: ['-DFMT_ENFORCE_COMPILE_STRING'],
  dependencies: [test_main_dep],
  override_options: ['warning_level=0'],
)
test('enforce-checks-test', enforce_checks_exe)

# ── Header-only tests ─────────────────────────────────────────────────────────
# These compile everything inline instead of linking the fmt library.
header_only_base_sources = [
  'test-main.cc',
  'gtest-extra.cc',
  'util.cc',
  '../src/os.cc',
]
header_only_deps = [gtest_dep, fmt_header_only_dep]
header_only_opts = ['warning_level=0']

format_impl_test_exe = executable('format-impl-test',
  sources: ['format-impl-test.cc', 'header-only-test.cc'] + header_only_base_sources,
  include_directories: test_inc,
  dependencies: header_only_deps,
  override_options: header_only_opts,
)
test('format-impl-test', format_impl_test_exe)

no_builtin_types_test_exe = executable('no-builtin-types-test',
  sources: ['no-builtin-types-test.cc'] + header_only_base_sources,
  include_directories: test_inc,
  dependencies: header_only_deps,
  override_options: header_only_opts,
)
test('no-builtin-types-test', no_builtin_types_test_exe)

scan_test_exe = executable('scan-test',
  sources: ['scan-test.cc'] + header_only_base_sources,
  include_directories: test_inc,
  dependencies: header_only_deps,
  override_options: header_only_opts,
)
test('scan-test', scan_test_exe)

unicode_test_extra = []
if host_machine.system() == 'windows' and cpp.get_id() == 'msvc'
  unicode_test_extra = ['/utf-8']
endif
unicode_test_exe = executable('unicode-test',
  sources: ['unicode-test.cc'] + header_only_base_sources,
  include_directories: test_inc,
  cpp_args: unicode_test_extra,
  dependencies: header_only_deps,
  override_options: header_only_opts,
)
test('unicode-test', unicode_test_exe)

# ── POSIX-specific tests (skipped on Windows) ─────────────────────────────────
if host_machine.system() != 'windows'
  posix_mock_test_exe = executable('posix-mock-test',
    sources: [
      'posix-mock-test.cc',
      '../src/format.cc',
      'test-main.cc',
      'gtest-extra.cc',
      'util.cc',
    ],
    include_directories: test_inc,
    dependencies: [gtest_dep],
    override_options: ['warning_level=0'],
  )
  test('posix-mock-test', posix_mock_test_exe)

  os_test_exe = executable('os-test',
    sources: 'os-test.cc',
    dependencies: [test_main_dep],
    override_options: ['warning_level=0'],
  )
  test('os-test', os_test_exe)
endif

# ── C API test ────────────────────────────────────────────────────────────────
c_test_exe = executable('c-test',
  sources: 'c-test.c',
  dependencies: [fmt_c_dep],
  override_options: ['warning_level=0'],
)
test('c-test', c_test_exe)

# ── Performance sanity (not a registered test, just build it) ─────────────────
executable('perf-sanity',
  sources: 'perf-sanity.cc',
  dependencies: [fmt_dep],
  override_options: ['warning_level=0'],
)
