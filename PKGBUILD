_pkgname=llama-cpp-turboquant
pkgname=llama-cpp-turboquant-hip
pkgver=feature.turboquant.kv.cache.b9966.f272689.0.gf27268914
pkgrel=1
pkgdesc="Fork do llama.cpp com TurboQuant+ (KV cache/quantização WHT-rotated), build HIP/ROCm para RDNA (gfx1201)"
arch=('x86_64')
url="https://github.com/TheTom/llama-cpp-turboquant"
license=('MIT')
depends=('rocm-hip-runtime' 'hipblas' 'rocblas' 'gcc-libs' 'glibc' 'curl')
makedepends=('cmake' 'git' 'ninja' 'rocm-hip-sdk')
provides=('llama.cpp')
conflicts=('llama.cpp' 'llama.cpp-hip')
source=("git+${url}.git#branch=feature/turboquant-kv-cache")
sha256sums=('SKIP')

# GPU alvo: RX 9070 XT (RDNA4) -> gfx1201
_amdgpu_targets="gfx1201"

pkgver() {
  cd "$srcdir/${_pkgname}"
  git describe --tags --long 2>/dev/null | sed 's/^v//;s/-/./g' || printf "0.3.0.r%s" "$(git rev-list --count HEAD)"
}

prepare() {
  cd "$srcdir/${_pkgname}"
  # garante que estamos na branch/tag desejada; ajuste se quiser fixar num release específico
  git submodule update --init --recursive
}

build() {
  cd "$srcdir/${_pkgname}"

  HIPCXX="$(hipconfig -l)/clang" \
  HIP_PATH="$(hipconfig -R)" \
  cmake -S . -B build -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DGGML_HIP=ON \
    -DAMDGPU_TARGETS="${_amdgpu_targets}" \
    -DGGML_HIP_ROCWMMA_FATTN=ON \
    -DBUILD_SHARED_LIBS=ON

  cmake --build build --config Release
}

package() {
  cd "$srcdir/${_pkgname}"
  DESTDIR="$pkgdir" cmake --install build
}
