# Multidimensional (2D and 3D) Image Augmentation for TensorFlow

_Note: this package is archived and no longer actively maintained on github. It
remains as a public snapshot of the internal project as of January 2021._

This package provides TensorFlow Ops for multidimensional volumetric image
augmentation.

## Install prerequities

This project usings the [bazel build
system](https://docs.bazel.build/versions/master/install.html)

## Build and test

To fetch the code, build it, and run tests:

```shell
git clone https://github.com/deepmind/multidim-image-augmentation.git
cd multidim-image-augmentation/
bazel test --python_version=py3 --config=nativeopt //...
```

Note bazel 0.24.0 made a lot of backward incompatible changes to default flag
values, that have not yet been resolved in this project and its dependencies.
In the meantime, you can disable with a few simple extra flags:

```shell
# Bazel >= 0.24.0
bazel test \
    --incompatible_disable_genrule_cc_toolchain_dependency=false \
    --incompatible_disable_legacy_cc_provider=false \
    --incompatible_disable_third_party_license_checking=false \
    --incompatible_no_transitive_loads=false \
    --incompatible_bzl_disallow_load_after_statement=false \
    --incompatible_disallow_load_labels_to_cross_package_boundaries=false \
    --config=nativeopt //...
```

To learn more about image augmentation, see the [primer](doc/index.md)

For simple API usage examples, see the python test code.

## Build a Python pip

More complex build process to ensure the custom_op .so file is built with the
same flags as the corresponding tensorflow package.

As a result we splice the custom_op build into the Tensorflow tree, and make a
few tweaks to get it to work.

Inspired by:
*   https://www.tensorflow.org/install/source#install_python_and_the_tensorflow_package_dependencies
*   https://github.com/tensorflow/custom-op
*   https://github.com/deepmind/multidim-image-augmentation/pull/31

Instructions for building with Python 3.7 and Tensorflow 1.15.2:

This has been tested inside the provided docker container. Mount host volumes
for exflitrating the built packages:

```shell
docker run -v LOCAL_PATH:CONTAINER_PATH -it tensorflow/tensorflow:custom-op-ubuntu16 /bin/bash
```

One time setup:

```shell
pip3.7 install "numpy<1.19.0"
pip3.7 install keras_preprocessing --no-deps

BAZEL_VERSION=0.26.1
wget https://github.com/bazelbuild/bazel/releases/download/$BAZEL_VERSION/bazel-$BAZEL_VERSION-installer-linux-x86_64.sh
chmod u+x bazel-$BAZEL_VERSION-installer-linux-x86_64.sh
./bazel-$BAZEL_VERSION-installer-linux-x86_64.sh
bazel version

TF_VERSION=1.15.2
wget https://github.com/tensorflow/tensorflow/archive/v$TF_VERSION.tar.gz
tar -xzf v$TF_VERSION.tar.gz

wget https://github.com/marcwilson-g/multidim-image-augmentation/archive/refs/heads/pip_build.zip
unzip pip_build.zip
cd multidim-image-augmentation-pip_build

find . -name "BUILD" | xargs sed -i 's/@org_tensorflow//g'
find . -name "BUILD" | xargs sed -i 's/protobuf_archive/com_google_protobuf/g'
# Remove int64 versions for now as it doesn't currently build. Re-instate if needed.
sed -i -E 's/(REG.*int64.*$)/\/\/\1/g' multidim_image_augmentation/kernels/apply_tabulated_functions_op.cc

cd ../tensorflow-$TF_VERSION
ln -s `readlink -f ../multidim-image-augmentation-pip_build/multidim_image_augmentation` .

./configure
```

Use the config defaults except for:

```
Please specify the location of python. [Default is /usr/bin/python]:
/usr/local/bin/python3.7
```

Perform the Build:

(this may take a LOOOOONG time)

```shell
bazel build --check_visibility=false --cxxopt=-D_GLIBCXX_USE_CXX11_ABI=0 //multidim_image_augmentation/pip:build_pip_pkg
bazel-bin/multidim_image_augmentation/pip/build_pip_pkg dist
```
