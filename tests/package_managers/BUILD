package(default_visibility = ["//visibility:public"])

load(
    "@bazel_tools//tools/build_rules:test_rules.bzl",
    "file_test",
    "rule_test",
)
load("//package_managers:download_pkgs.bzl", "download_pkgs")
load("//package_managers:install_pkgs.bzl", "install_pkgs")
load("//package_managers:apt_key.bzl", "add_apt_key")
load("@io_bazel_rules_docker//contrib:test.bzl", "container_test")
load("@io_bazel_rules_docker//contrib:compare_ids_test.bzl", "compare_ids_test")
load("//package_managers:bootstrap_image.bzl", "bootstrap_image_macro")

download_pkgs(
    name = "test_download_pkgs",
    image_tar = "//ubuntu:ubuntu_16_0_4_vanilla.tar",
    packages = [
        "curl",
        "netbase",
    ],
)

rule_test(
    name = "test_download_pkgs_rule",
    generates = [
        "test_download_pkgs",
    ],
    rule = "test_download_pkgs",
)

file_test(
    name = "test_download_pkgs_docker_run",
    file = ":test_download_pkgs",
    regexp = "image_id.* ubuntu/ubuntu_16_0_4_vanilla.tar)$",
)

file_test(
    name = "test_download_pkgs_docker_cp",
    file = ":test_download_pkgs",
    regexp = ".*docker cp .*:test_download_pkgs.tar tests/package_managers/test_download_pkgs.tar.*",
)

sh_test(
    name = "download_pkgs_run_test",
    srcs = ["download_pkgs_run_test.sh"],
    data = [":test_download_pkgs.tar"],
)

add_apt_key(
    name = "ubuntu_gpg_image",
    image = "//ubuntu:ubuntu_16_0_4_vanilla.tar",
    keys = [
        "@bazel_gpg//file",
    ],
)

download_pkgs(
    name = "test_download_pkgs_with_additional_repos",
    additional_repos = [
        "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8",
    ],
    image_tar = ":ubuntu_gpg_image.tar",
    packages = [
        "bazel",
    ],
)

sh_test(
    name = "download_pkgs_with_additional_repos_run_test",
    srcs = ["download_pkgs_with_additional_repos_run_test.sh"],
    data = [":test_download_pkgs_with_additional_repos.tar"],
)

install_pkgs(
    name = "test_install_pkgs",
    image_tar = "//ubuntu:ubuntu_16_0_4_vanilla.tar",
    installables_tar = ":test_download_pkgs.tar",
    output_image_name = "test_install_pkgs",
)

rule_test(
    name = "test_install_pkgs_rule",
    generates = [
        "test_install_pkgs.tar",
    ],
    rule = "test_install_pkgs",
)

add_apt_key(
    name = "gpg_image",
    image = "//debian/reproducible:debian9.tar",
    keys = [
        "@bazel_gpg//file",
        "@launchpad_openjdk_gpg//file",
    ],
)

container_test(
    name = "gpg_test",
    configs = [":gpg_test.yaml"],
    image = ":gpg_image",
)

bootstrap_image_macro(
    name = "bootstrap_ubuntu",
    date = "20181126",
    image_tar = "//ubuntu:ubuntu_16_0_4_vanilla.tar",
    output_image_name = "ubuntu",
    packages = [
        "curl",
        "netbase",
    ],
    store_location = "ubuntu/16_0_4/builds",
)

rule_test(
    name = "test_bootstrap_ubuntu",
    generates = [
        "bootstrap_ubuntu-layer.tar",
    ],
    rule = "bootstrap_ubuntu",
)

# Testing reproducibility of install_pkgs

download_pkgs(
    name = "download_pkg_git",
    image_tar = "//ubuntu:ubuntu_16_0_4_vanilla.tar",
    packages = [
        "git",
    ],
)

install_pkgs(
    name = "install_git_for_reproducibility_1",
    image_tar = "//ubuntu:ubuntu_16_0_4_vanilla.tar",
    installables_tar = ":download_pkg_git.tar",
    output_image_name = "test_install_pkgs_duplicate",
)

install_pkgs(
    name = "install_git_for_reproducibility_2",
    image_tar = "//ubuntu:ubuntu_16_0_4_vanilla.tar",
    installables_tar = ":download_pkg_git.tar",
    output_image_name = "test_install_pkgs_duplicate",
)

compare_ids_test(
    name = "install_pkgs_reproducibility_test",
    images = [
        ":install_git_for_reproducibility_1",
        ":install_git_for_reproducibility_2",
    ],
)
