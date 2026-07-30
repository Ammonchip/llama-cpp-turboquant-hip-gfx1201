# llama-cpp-turboquant-hip

PKGBUILD para o [llama-cpp-turboquant](https://github.com/TheTom/llama-cpp-turboquant) (fork do
[llama.cpp](https://github.com/ggml-org/llama.cpp) com TurboQuant+, quantização de KV cache
via rotação WHT), compilado com backend **HIP/ROCm** para GPUs AMD RDNA4.

Testado em:

- **GPU:** AMD Radeon RX 9070 XT (RDNA4, `gfx1201`)
- **CPU:** AMD Ryzen 7 7700
- **RAM:** 32 GB
- **SO:** CachyOS (Arch-based)
- **DE:** KDE Plasma
- **Shell:** fish

## Branch usada

Este PKGBUILD puxa o branch **`feature/turboquant-kv-cache`**, onde vivem as otimizações de
TurboQuant+ ainda não mescladas no `main`. Se o upstream já tiver mesclado essas mudanças na
branch principal quando você ler isto, ajuste o `source=()` no PKGBUILD de acordo.

## Diferenças em relação ao llama.cpp upstream

- Adiciona quantização de KV cache com rotação Walsh-Hadamard (TurboQuant+), reduzindo uso de
  VRAM em contextos longos com perda de qualidade menor que quantização KV cache tradicional.
- Mesma base de código/arquitetura do `ggml`/`llama.cpp` — os binários resultantes
  (`llama-cli`, `llama-server`, `llama-bench`, etc.) funcionam da mesma forma.
- Por ser um fork mais experimental, pode exigir versões de ROCm mais recentes que o upstream
  para compilar sem erros (ver seção de dependências abaixo).

## Dependências para compilar

### Runtime (`depends`)

```
rocm-hip-runtime
hipblas
rocblas
gcc-libs
glibc
curl
```

### Build (`makedepends`)

```
cmake
git
ninja
rocm-hip-sdk
rocwmma
```

> **`rocwmma` é obrigatório** se você mantiver `GGML_HIP_ROCWMMA_FATTN=ON` no PKGBUILD (ativa
> flash-attention otimizado via WMMA). Sem essa lib instalada, o build falha com:
> ```
> fatal error: 'rocwmma/rocwmma-version.hpp' file not found
> ```
> Se preferir não instalar o rocWMMA, mude a flag para `OFF` no PKGBUILD — o resto compila
> normalmente, só perde essa otimização específica de prefill.

Instalação das dependências no CachyOS/Arch:

```bash
sudo pacman -S cmake git ninja rocm-hip-sdk rocm-hip-runtime hipblas rocblas rocwmma curl
```

## Ajustando para sua GPU

O target de arquitetura está fixo em `gfx1201` (RDNA4 / RX 9070 XT) na variável `_amdgpu_targets`
do PKGBUILD. Para outras GPUs AMD, troque pelo código correspondente (`gfx1100` para RX 7900
series, `gfx1030` para RX 6800/6900, etc). Para descobrir o código da sua GPU:

```bash
rocminfo | grep gfx
```

## Como compilar e instalar

```bash
git clone <url-deste-repo>
cd llama-cpp-turboquant-hip
makepkg -si
```

## Como atualizar

O PKGBUILD usa `source=("git+...")`, então o `makepkg` puxa os commits novos do branch
automaticamente:

```bash
makepkg -si
```

Para uma atualização "limpa" (sem cache de builds anteriores):

```bash
rm -rf src/ pkg/
makepkg -si
```

## Testando a instalação

Verificar detecção da GPU e offload:

```bash
llama-cli -hf Qwen/Qwen2.5-0.5B-Instruct-GGUF -ngl 99 --verbose -p "teste"
```

Procure no log por `ggml_cuda_init: found ... ROCm devices` e `offloaded N/N layers to GPU`.

Benchmark comparando com/sem flash-attention (rocWMMA):

```bash
llama-bench -hf Qwen/Qwen2.5-0.5B-Instruct-GGUF -ngl 99 -fa 0,1 --device ROCm0
```

## Desinstalando

Como a instalação é gerenciada pelo pacman, remover é direto:

```bash
sudo pacman -R llama-cpp-turboquant-hip
```

## Licença

MIT, seguindo a licença do projeto upstream ([TheTom/llama-cpp-turboquant](https://github.com/TheTom/llama-cpp-turboquant)).
