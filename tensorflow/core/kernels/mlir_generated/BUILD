# Generates CUDA kernels using MLIR codegen.

load(
    "//tensorflow/core/kernels/mlir_generated:build_defs.bzl",
    "gen_kernel_library",
    "if_mlir_generated_gpu_kernels_enabled",
    "if_mlir_unranked_kernels_enabled",
)
load(
    "//tensorflow:tensorflow.bzl",
    "if_cuda_or_rocm",
)
load("//tensorflow:tensorflow.bzl", "tf_cuda_cc_test")  # buildifier: disable=same-origin-load
load("//tensorflow:tensorflow.bzl", "tf_kernel_library")  # buildifier: disable=same-origin-load
load(
    "//tensorflow/core/platform:build_config_root.bzl",
    "tf_cuda_tests_tags",
)

package(
    default_visibility = [
        "//tensorflow/core/kernels:__subpackages__",
    ],
    licenses = ["notice"],  # Apache 2.0
)

config_setting(
    name = "mlir_generated_gpu_kernels_disabled",
    define_values = {
        "tensorflow_enable_mlir_generated_gpu_kernels": "0",
    },
)

config_setting(
    name = "mlir_use_unranked_kernels",
    define_values = {"enable_unranked_kernels": "1"},
)

filegroup(
    name = "unary_kernel_srcs",
    srcs = if_mlir_unranked_kernels_enabled(
        [
            "unranked_op_gpu_abs.cc",
            "unranked_op_gpu_ceil.cc",
            "unranked_op_gpu_tanh.cc",
            "unranked_op_gpu_base.h",
            "unranked_op_gpu_base.cc",
        ],
        [
            "cwise_op_gpu_abs.cc",
            "cwise_op_gpu_base.cc",
            "cwise_op_gpu_base.h",
            "cwise_op_gpu_tanh.cc",
        ],
    ),
)

cc_library(
    name = "unranked_kernel_deps",
    deps =
        [
            ":abs_unranked_kernels",
            ":addv2_unranked_kernels",
            ":ceil_unranked_kernels",
            ":tanh_unranked_kernels",
            "//tensorflow/compiler/mlir/tools/kernel_gen:tf_cuda_runtime_wrappers",
            "//tensorflow/compiler/mlir/tools/kernel_gen:tf_framework_c_interface",
        ],
)

cc_library(
    name = "ranked_kernel_deps",
    deps =
        [
            ":abs_kernels",
            ":tanh_kernels",
        ],
)

cc_library(
    name = "kernel_deps",
    deps = if_mlir_unranked_kernels_enabled(
        [
            ":unranked_kernel_deps",
        ],
        [
            ":ranked_kernel_deps",
        ],
    ),
)

tf_kernel_library(
    name = "cwise_unary_op",
    # Technically these source files don't need --config=cuda or --config=rocm,
    # but we want to avoid building them if they are not needed.
    srcs = if_cuda_or_rocm([":unary_kernel_srcs"]),
    tags = ["no_rocm"],
    deps = if_cuda_or_rocm(
        [
            ":kernel_deps",
            "@com_google_absl//absl/strings",
            "@com_google_absl//absl/synchronization",
            "@com_google_absl//absl/types:span",
            "//third_party/eigen3",
            "//tensorflow/core:framework",
            "//tensorflow/core:lib",
            "//tensorflow/core/platform:stream_executor",
        ],
    ),
)

tf_kernel_library(
    name = "cwise_binary_op",
    # Technically these source files don't need --config=cuda or --config=rocm,
    # but we want to avoid building them if they are not needed.
    srcs = if_cuda_or_rocm(["unranked_gpu_add.cc"]),
    tags = [
        "manual",
        "no_rocm",
    ],
    deps = if_cuda_or_rocm([
        ":unranked_kernel_deps",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/synchronization",
        "@com_google_absl//absl/types:span",
        "//third_party/eigen3",
        "//tensorflow/core:framework",
        "//tensorflow/core:lib",
        "//tensorflow/core/platform:stream_executor",
    ]),
)

tf_kernel_library(
    name = "cwise_op",
    srcs = [],
    tags = ["no_rocm"],
    deps = [
        ":cwise_unary_op",
    ] + if_mlir_unranked_kernels_enabled(
        [
            ":cwise_binary_op",
        ],
    ),
)

tf_cuda_cc_test(
    name = "gpu_tanh_test",
    size = "small",
    srcs = if_mlir_generated_gpu_kernels_enabled(["gpu_tanh_test.cc"]),
    tags = tf_cuda_tests_tags() + [
        "no_cuda_asan",  # TODO(b/171341759): re-enable.
    ],
    deps = [
        "//tensorflow/core:framework",
        "//tensorflow/core:framework_internal",
        "//tensorflow/core:tensorflow",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
        "//tensorflow/core/kernels:cwise_op",
        "//tensorflow/core/kernels:ops_testutil",
    ],
)

tf_cuda_cc_test(
    name = "gpu_abs_test",
    size = "small",
    srcs = if_mlir_generated_gpu_kernels_enabled(["gpu_abs_test.cc"]),
    tags = tf_cuda_tests_tags() + [
        "no_cuda_asan",  # TODO(b/171341759): re-enable.
    ],
    deps = [
        "//tensorflow/core:framework",
        "//tensorflow/core:framework_internal",
        "//tensorflow/core:tensorflow",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
        "//tensorflow/core/kernels:cwise_op",
        "//tensorflow/core/kernels:ops_testutil",
    ],
)

tf_cuda_cc_test(
    name = "gpu_add_test",
    size = "small",
    srcs = if_mlir_generated_gpu_kernels_enabled(["gpu_add_test.cc"]),
    tags = tf_cuda_tests_tags(),
    deps = [
        "//tensorflow/core:framework",
        "//tensorflow/core:framework_internal",
        "//tensorflow/core:tensorflow",
        "//tensorflow/core:test",
        "//tensorflow/core:test_main",
        "//tensorflow/core:testlib",
        "//tensorflow/core/common_runtime:device",
        "//tensorflow/core/common_runtime:device_factory",
        "//tensorflow/core/kernels:cwise_op",
        "//tensorflow/core/kernels:ops_testutil",
    ],
)
# TODO(b/160731748): Re-enable when it works again.
# gen_kernel_library(
#     name = "bias_add",
#     same_shape = "0,2",
#     tile_size = "16x16",
#     types = [
#         "f16",
#         "f32",
#         "f64",
#     ],
# )

# TODO(b/160190568): Re-enable when it works again.
# gen_kernel_library(
#     name = "relu",
#     same_shape = "0,1",
#     tile_size = "256",
#     types = [
#         "f16",
#         "f32",
#         "f64",
#     ],
# )

gen_kernel_library(
    name = "abs",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
        "i32",
        "i64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "ceil",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "conj",
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "c64",
        "c128",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "cos",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "exp",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "floor",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "imag",
    tile_size = "256",
    types = [
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "invert",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "i8",
        "i16",
        "i32",
        "i64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "isfinite",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "log",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "logicalnot",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = ["i1"],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "neg",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "real",
    tile_size = "256",
    types = [
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "rsqrt",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "sign",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        # TODO(b/162577610): Add bf16, c64 and c128.
        "f16",
        "f32",
        "f64",
        "i32",
        "i64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "sqrt",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "tanh",
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)

gen_kernel_library(
    name = "addv2",
    generate_ranked = False,
    generate_unranked = True,
    same_shape = "0,1",
    tile_size = "256",
    types = [
        "f16",
        "f32",
        "f64",
    ],
    unroll_factors = "4",
)
