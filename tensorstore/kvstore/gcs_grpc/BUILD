load("//bazel:tensorstore.bzl", "tensorstore_cc_library", "tensorstore_cc_test")
load("@bazel_skylib//rules:common_settings.bzl", "bool_flag")

package(default_visibility = ["//visibility:public"])

licenses(["notice"])

# To enable debug checks, specify:
# bazel build --//tensorstore/kvstore/gcs_grpc:debug
bool_flag(
    name = "debug",
    build_setting_default = False,
)

config_setting(
    name = "debug_setting",
    flag_values = {
        ":debug": "True",
    },
    visibility = ["//visibility:private"],
)

DEBUG_LOCAL_DEFINES = select({
    ":debug_setting": ["TENSORSTORE_GCS_GRPC_DEBUG"],
    "//conditions:default": [],
})

tensorstore_cc_library(
    name = "gcs_grpc",
    srcs = ["gcs_grpc.cc"],
    local_defines = DEBUG_LOCAL_DEFINES,
    deps = [
        ":get_credentials",
        ":storage_stub_pool",
        "//tensorstore:context",
        "//tensorstore/internal:context_binding",
        "//tensorstore/internal:data_copy_concurrency_resource",
        "//tensorstore/internal:intrusive_ptr",
        "//tensorstore/internal:retry",
        "//tensorstore/internal:schedule_at",
        "//tensorstore/internal:uri_utils",
        "//tensorstore/internal/cache_key",
        "//tensorstore/internal/grpc:utils",
        "//tensorstore/internal/json_binding",
        "//tensorstore/internal/json_binding:absl_time",
        "//tensorstore/internal/json_binding:bindable",
        "//tensorstore/internal/metrics",
        "//tensorstore/kvstore",
        "//tensorstore/kvstore:byte_range",
        "//tensorstore/kvstore:generation",
        "//tensorstore/kvstore:key_range",
        "//tensorstore/kvstore/gcs:gcs_resource",
        "//tensorstore/kvstore/gcs:validate",
        "//tensorstore/proto:proto_util",
        "//tensorstore/serialization",
        "//tensorstore/serialization:absl_time",
        "//tensorstore/util:executor",
        "//tensorstore/util:future",
        "//tensorstore/util:quote_string",
        "//tensorstore/util:result",
        "//tensorstore/util:str_cat",
        "//tensorstore/util/execution",
        "//tensorstore/util/execution:any_receiver",
        "//tensorstore/util/garbage_collection",
        "@com_github_grpc_grpc//:grpc++",
        "@com_google_absl//absl/base:core_headers",
        "@com_google_absl//absl/crc:crc32c",
        "@com_google_absl//absl/log:absl_log",
        "@com_google_absl//absl/status",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/strings:cord",
        "@com_google_absl//absl/strings:str_format",
        "@com_google_absl//absl/synchronization",
        "@com_google_absl//absl/time",
        "@com_google_googleapis//google/storage/v2:storage_cc_grpc",
        "@com_google_googleapis//google/storage/v2:storage_cc_proto",
        "@com_google_protobuf//:protobuf",
    ],
    alwayslink = 1,
)

tensorstore_cc_test(
    name = "gcs_grpc_test",
    srcs = ["gcs_grpc_test.cc"],
    deps = [
        ":gcs_grpc",
        ":mock_storage_service",
        "//tensorstore:context",
        "//tensorstore/internal:flat_cord_builder",
        "//tensorstore/internal/grpc:grpc_mock",
        "//tensorstore/internal/grpc:utils",
        "//tensorstore/kvstore",
        "//tensorstore/kvstore:byte_range",
        "//tensorstore/kvstore:generation",
        "//tensorstore/kvstore:key_range",
        "//tensorstore/kvstore:test_util",
        "//tensorstore/proto:parse_text_proto_or_die",
        "//tensorstore/proto:protobuf_matchers",
        "//tensorstore/util:future",
        "//tensorstore/util:status_testutil",
        "//tensorstore/util/execution",
        "//tensorstore/util/execution:sender_testutil",
        "@com_google_absl//absl/log:absl_log",
        "@com_google_absl//absl/status",
        "@com_google_absl//absl/strings:cord",
        "@com_google_absl//absl/strings:str_format",
        "@com_google_absl//absl/synchronization",
        "@com_google_absl//absl/time",
        "@com_google_googleapis//google/storage/v2:storage_cc_proto",
        "@com_google_googletest//:gtest_main",
    ],
)

tensorstore_cc_library(
    name = "mock_storage_service",
    testonly = 1,
    hdrs = ["mock_storage_service.h"],
    visibility = ["//visibility:private"],
    deps = [
        "//tensorstore/internal/grpc:grpc_mock",
        "@com_github_grpc_grpc//:grpc++",
        "@com_google_googleapis//google/storage/v2:storage_cc_grpc",
        "@com_google_googleapis//google/storage/v2:storage_cc_proto",
        "@com_google_protobuf//:protobuf",
    ],
)

tensorstore_cc_test(
    name = "gcs_grpc_testbench_test",
    size = "medium",
    srcs = [
        "gcs_grpc_testbench_test.cc",
    ],
    args = ["--testbench_binary=$(location @com_google_storagetestbench//:rest_server)"],
    data = ["@com_google_storagetestbench//:rest_server"],
    flaky = 1,
    tags = [
        "requires-net:loopback",
        "skip-cmake",  # storage_testbench is python, which doesn't work in bazel_to_cmake yet.
        "skip-darwin",  # the python binary doesn't find dependencies on darwin.
    ],
    deps = [
        ":gcs_grpc",
        "//tensorstore/internal:no_destructor",
        "//tensorstore/internal:thread",
        "//tensorstore/kvstore",
        "//tensorstore/kvstore:generation",
        "//tensorstore/kvstore:test_util",
        "//tensorstore/kvstore/gcs:gcs_testbench",
        "//tensorstore/util:future",
        "//tensorstore/util:status_testutil",
        "@com_google_absl//absl/log:absl_log",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/strings:cord",
        "@com_google_googletest//:gtest_main",
    ],
)

tensorstore_cc_library(
    name = "get_credentials",
    srcs = ["get_credentials.cc"],
    hdrs = ["get_credentials.h"],
    visibility = ["//visibility:private"],
    deps = [
        "@com_github_grpc_grpc//:grpc++",
        "@com_google_absl//absl/log:absl_log",
        "@com_google_absl//absl/strings",
    ],
)

tensorstore_cc_library(
    name = "storage_stub_pool",
    srcs = ["storage_stub_pool.cc"],
    hdrs = ["storage_stub_pool.h"],
    local_defines = DEBUG_LOCAL_DEFINES,
    visibility = ["//visibility:private"],
    deps = [
        "//tensorstore/internal:env",
        "//tensorstore/internal:no_destructor",
        "@com_github_grpc_grpc//:grpc++",
        "@com_github_grpc_grpc//:grpc_security_base",
        "@com_google_absl//absl/base:core_headers",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/flags:flag",
        "@com_google_absl//absl/log:absl_log",
        "@com_google_absl//absl/strings",
        "@com_google_absl//absl/strings:str_format",
        "@com_google_absl//absl/synchronization",
        "@com_google_absl//absl/time",
        "@com_google_googleapis//google/storage/v2:storage_cc_grpc",
    ],
)
