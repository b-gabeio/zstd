project('zstd', 'c', license: 'BSD')

libm = meson.get_compiler('c').find_library('m', required: true)

lib_dir = join_paths(meson.source_root(), '..', '..', 'lib')
common_dir = join_paths(lib_dir, 'common')
compress_dir = join_paths(lib_dir, 'compress')
decompress_dir = join_paths(lib_dir, 'decompress')
dictbuilder_dir = join_paths(lib_dir, 'dictBuilder')
deprecated_dir = join_paths(lib_dir, 'deprecated')

libzstd_srcs = [join_paths(common_dir, 'entropy_common.c'), join_paths(common_dir, 'fse_decompress.c'), join_paths(common_dir, 'threading.c'), join_paths(common_dir, 'pool.c'), join_paths(common_dir, 'zstd_common.c'), join_paths(common_dir, 'error_private.c'), join_paths(common_dir, 'xxhash.c'), join_paths(compress_dir, 'fse_compress.c'), join_paths(compress_dir, 'huf_compress.c'), join_paths(compress_dir, 'zstd_compress.c'), join_paths(compress_dir, 'zstdmt_compress.c'), join_paths(decompress_dir, 'huf_decompress.c'), join_paths(decompress_dir, 'zstd_decompress.c'), join_paths(dictbuilder_dir, 'cover.c'), join_paths(dictbuilder_dir, 'divsufsort.c'), join_paths(dictbuilder_dir, 'zdict.c'), join_paths(deprecated_dir, 'zbuff_common.c'), join_paths(deprecated_dir, 'zbuff_compress.c'), join_paths(deprecated_dir, 'zbuff_decompress.c')]

libzstd_includes = [include_directories(common_dir, dictbuilder_dir, compress_dir, lib_dir)]

if get_option('legacy_support')
    message('Enabling legacy support')
    libzstd_cflags = ['-DZSTD_LEGACY_SUPPORT=4']

    legacy_dir = join_paths(lib_dir, 'legacy')
    libzstd_includes += [include_directories(legacy_dir)]
    libzstd_srcs += [join_paths(legacy_dir, 'zstd_v01.c'), join_paths(legacy_dir, 'zstd_v02.c'), join_paths(legacy_dir, 'zstd_v03.c'), join_paths(legacy_dir, 'zstd_v04.c'), join_paths(legacy_dir, 'zstd_v05.c'), join_paths(legacy_dir, 'zstd_v06.c'), join_paths(legacy_dir, 'zstd_v07.c')]
else
    libzstd_cflags = []
endif

if get_option('multithread')
    message('Enabling multi-threading support')
    add_global_arguments('-DZSTD_MULTITHREAD', language: 'c')
    libzstd_deps = [dependency('threads')]
else
    libzstd_deps = []
endif

libzstd = library('zstd',
                  libzstd_srcs,
                  include_directories: libzstd_includes,
                  c_args: libzstd_cflags,
                  dependencies: libzstd_deps,
                  install: true)

programs_dir = join_paths(meson.source_root(), '..', '..', 'programs')

zstd = executable('zstd',
                  join_paths(programs_dir, 'bench.c'), join_paths(programs_dir, 'datagen.c'), join_paths(programs_dir, 'dibio.c'), join_paths(programs_dir, 'fileio.c'), join_paths(programs_dir, 'zstdcli.c'),
                  include_directories: libzstd_includes,
                  c_args: ['-DZSTD_NODICT', '-DZSTD_NOBENCH'],
                  link_with: libzstd,
                  install: true)

tests_dir = join_paths(meson.source_root(), '..', '..', 'tests')
datagen_c = join_paths(programs_dir, 'datagen.c')
test_includes = libzstd_includes + [include_directories(programs_dir)]

fullbench = executable('fullbench',
                       datagen_c, join_paths(tests_dir, 'fullbench.c'),
                       include_directories: test_includes,
                       link_with: libzstd)
test('fullbench', fullbench)

fuzzer = executable('fuzzer',
                    datagen_c, join_paths(tests_dir, 'fuzzer.c'),
                    include_directories: test_includes,
                    link_with: libzstd)
test('fuzzer', fuzzer)

if target_machine.system() != 'windows'
    paramgrill = executable('paramgrill',
                            datagen_c, join_paths(tests_dir, 'paramgrill.c'),
                            include_directories: test_includes,
                            link_with: libzstd,
                            dependencies: libm)
    test('paramgrill', paramgrill)

    datagen = executable('datagen',
                         datagen_c, join_paths(tests_dir, 'datagencli.c'),
                         include_directories: test_includes,
                         link_with: libzstd)
endif
