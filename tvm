#!/usr/bin/env python3

"""CLI application to install and use specific Terraform versions."""

import glob
import os
import platform
import shutil
import sys
from zipfile import ZipFile

import fire
import requests


def get_osinfo() -> str:
    """Get OS and architecture information"""

    user_os = platform.system()
    user_arch = platform.machine()

    if user_os in ("Darwin", "Linux"):
        user_os = user_os.lower()
    else:
        print(f"{user_os} is not supported. Quitting.")
        sys.exit(1)

    if user_arch in ("arm64", "aarch64"):
        user_arch = "arm64"
    else:
        user_arch = "amd64"

    return f"{user_os}_{user_arch}"


def install_terraform(*versions):
    """Install a specific version (or multiple versions) of Terraform"""

    osinfo = get_osinfo()
    temp_dir = f"{TVM_PATH}/tmp"

    for version in versions:
        file_url = f"https://releases.hashicorp.com/terraform/{version}/terraform_{version}_{osinfo}.zip"
        zip_file = f"{temp_dir}/terraform_{version}_{osinfo}.zip"
        file_src = f"{temp_dir}/terraform"
        file_dst = f"{TVM_PATH}/terraform_{version}_{osinfo}"

        if os.path.isfile(file_dst):
            download_prompt = input(
                f"Terraform v{version} already exists. Do you want to download it again? [y/N]: "
            )
            if not download_prompt.lower() in ("y", "yes"):
                print("Quitting.")
                sys.exit(0)

        if not os.path.isdir(temp_dir):
            os.mkdir(temp_dir)
        else:
            try:
                shutil.rmtree(temp_dir)
                os.mkdir(temp_dir)
            except OSError as err:
                print(f"Error: {temp_dir} : {err.strerror}")

        print(f"Downloading from {file_url}...")
        resp = requests.get(file_url, timeout=30)

        if resp.status_code == 403:
            print("ERROR: Invalid Terraform version.")
            sys.exit(1)
        elif resp.status_code != 200:
            print(f"ERROR: Request not successful. Response code: {resp.status_code}")
            sys.exit(1)

        file_data = resp.content
        with open(zip_file, "wb") as file:
            file.write(file_data)

        print("Decompressing zip file...")
        with ZipFile(zip_file, "r") as zip_obj:
            zip_obj.extractall(temp_dir)

        # ZipFile does not preserve file permissions
        os.chmod(file_src, 0o755)

        print(f"Moving the binary to {TVM_PATH}")
        shutil.move(file_src, file_dst)

        print("Cleaning up temp directory...")
        try:
            shutil.rmtree(temp_dir)
        except OSError as err:
            print(f"Error: {temp_dir} : {err.strerror}")


def use_terraform(version: str):
    """Symlink a specific Terraform version"""

    osinfo = get_osinfo()
    terraform_src = f"{TVM_PATH}/terraform_{version}_{osinfo}"
    terraform_dst = f"{TVM_PATH}/terraform"

    if not os.path.isfile(terraform_src):
        download_prompt = input(
            f"Terraform v{version} cannot be found locally. Do you want to download? [Y/n]: "
        )
        if download_prompt.lower() in ("n", "no"):
            print("Quitting.")
            sys.exit(0)
        install_terraform(version)

    if os.path.isfile(terraform_dst):
        print("Removing existing symbolic link...")
        os.remove(terraform_dst)

    print(f"Creating a symbolic link for {terraform_src} to {terraform_dst}")
    os.symlink(terraform_src, terraform_dst)

    user_path = os.environ["PATH"]

    if not TVM_PATH in user_path:
        print("\nWarning: tvm directory is not in PATH.")
        print("Please add tvm directory to the path to use Terraform binaries:")
        print(f'\texport PATH="{TVM_PATH}:$PATH"')


def remove_unused():
    """Remove non-active Terraform binaries"""

    filelist = glob.glob(f"{TVM_PATH}/terraform_*")
    active_tf_file = f"{TVM_PATH}/terraform"

    if os.path.isfile(active_tf_file) and os.path.islink(active_tf_file):
        symlinked_file = os.path.realpath(active_tf_file)

    for item in filelist:
        if item == symlinked_file:
            continue
        print(f"Removing: {item}")
        os.remove(item)


def remove_all():
    """Remove all Terraform binaries"""

    filelist = glob.glob(f"{TVM_PATH}/terraform*")

    for item in filelist:
        print(f"Removing: {item}")
        os.remove(item)


def list_versions():
    """Print Terraform versions managed by tvm as a table"""

    filelist = sorted(glob.glob(f"{TVM_PATH}/terraform_*"), reverse=True)
    active_tf_file = f"{TVM_PATH}/terraform"

    if os.path.isfile(active_tf_file) and os.path.islink(active_tf_file):
        active_index = filelist.index(os.path.realpath(active_tf_file))

    basename_list = [os.path.basename(item) for item in filelist]

    active_list = [""] * len(basename_list)

    if "active_index" in dir():
        active_list[active_index] = "  *"

    version_table = list(zip(active_list, basename_list))

    print(f"{'ACTIVE':<8} {'VERSION':<15}")

    for item in version_table:
        is_active, version = item
        print(f"{is_active:<8} {version:<10}")


TVM_PATH = os.path.expanduser("~/.tvm")

if not os.path.isdir(TVM_PATH):
    print(f'tvm directory does not exist. Creating the directory "{TVM_PATH}"...')
    os.mkdir(TVM_PATH)
    print("Please add tvm directory to the path to use Terraform binaries:")
    print(f'\texport PATH="{TVM_PATH}:$PATH"')

if __name__ == "__main__":
    fire.Fire(
        {
            "install": install_terraform,
            "use": use_terraform,
            "remove": {
                "unused": remove_unused,
                "all": remove_all,
            },
            "list": list_versions,
        }
    )
