# Build for the undefined behavior sanitizer runtime support library.

set(UBSAN_MINIMAL_SOURCES
  ubsan_minimal_handlers.cc
  )

include_directories(..)

set(UBSAN_CFLAGS ${SANITIZER_COMMON_CFLAGS})
append_rtti_flag(OFF UBSAN_CFLAGS)

set(UBSAN_STANDALONE_CFLAGS ${SANITIZER_COMMON_CFLAGS})
append_rtti_flag(OFF UBSAN_STANDALONE_CFLAGS)

add_compiler_rt_component(ubsan-minimal)

# Common parts of UBSan runtime.
add_compiler_rt_object_libraries(RTUbsan_minimal
  OS ${SANITIZER_COMMON_SUPPORTED_OS}
  ARCHS ${UBSAN_COMMON_SUPPORTED_ARCH}
  SOURCES ${UBSAN_MINIMAL_SOURCES} CFLAGS ${UBSAN_CFLAGS})


if(COMPILER_RT_HAS_UBSAN_MINIMAL)
  # Initializer of standalone UBSan runtime.

  # Standalone UBSan runtimes.
  add_compiler_rt_runtime(clang_rt.ubsan_minimal
    STATIC
    OS ${SANITIZER_COMMON_SUPPORTED_OS}
    ARCHS ${UBSAN_SUPPORTED_ARCH}
    OBJECT_LIBS RTUbsan_minimal
    CFLAGS ${UBSAN_CFLAGS}
    PARENT_TARGET ubsan-minimal)

  add_compiler_rt_runtime(clang_rt.ubsan_minimal
    SHARED
    OS ${SANITIZER_COMMON_SUPPORTED_OS}
    ARCHS ${UBSAN_SUPPORTED_ARCH}
    OBJECT_LIBS RTUbsan_minimal
    CFLAGS ${UBSAN_CFLAGS}
    LINK_LIBS ${UBSAN_DYNAMIC_LIBS}
    PARENT_TARGET ubsan-minimal)

  if (UNIX AND NOT APPLE)
    set(ARCHS_FOR_SYMBOLS ${UBSAN_SUPPORTED_ARCH})
    list(REMOVE_ITEM ARCHS_FOR_SYMBOLS i386 i686)
    add_sanitizer_rt_symbols(clang_rt.ubsan_minimal
      ARCHS ${ARCHS_FOR_SYMBOLS}
      PARENT_TARGET ubsan-minimal
      EXTRA ubsan.syms.extra)
  endif()
endif()
