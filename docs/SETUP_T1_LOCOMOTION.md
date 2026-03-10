# Setup Guide: T1 Locomotion Task

Guia completo para rodar o treinamento de locomoção do Booster T1 em qualquer máquina.

---

## Requisitos

- Ubuntu 22.04 ou 24.04
- GPU NVIDIA com driver >= 525 (testado com RTX 5070)
- CUDA 12+
- Miniconda ou Anaconda
- ~15 GB de espaço em disco

---

## 1. Estrutura de pastas esperada

Coloque tudo dentro de uma mesma pasta raiz (ex: `~/Desktop/booster_train/`):

```
booster_train/          ← pasta raiz
├── IsaacLab/           ← repo do Isaac Lab
├── booster_assets/     ← assets do robô (URDFs, meshes)
└── booster_train/      ← este repo (tasks de treinamento)
```

---

## 2. Instalar Isaac Lab com conda

Siga o [guia oficial](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/conda_installation.html) ou os passos abaixo:

```bash
# Clone o Isaac Lab
cd ~/Desktop/booster_train
git clone https://github.com/isaac-sim/IsaacLab.git

# Crie o ambiente conda (Isaac Sim 5.x requer Python 3.11)
cd IsaacLab
conda create -n isaaclab_env python=3.11
conda activate isaaclab_env

# Instale o Isaac Sim (isso baixa ~10GB, pode demorar)
./isaaclab.sh --install
```

> Se o Isaac Sim já foi instalado manualmente em outro lugar, você pode criar um symlink:
> `ln -s /caminho/para/isaac_sim IsaacLab/_isaac_sim`

---

## 3. Corrigir o script de ativação do conda

O conda precisa saber onde está o Isaac Lab para configurar o `PYTHONPATH` corretamente.

Edite o arquivo:
```
~/.conda/envs/isaaclab_env/etc/conda/activate.d/setenv.sh
```
ou
```
~/miniconda3/envs/isaaclab_env/etc/conda/activate.d/setenv.sh
```

Conteúdo correto (ajuste o caminho se necessário):

```bash
#!/usr/bin/env bash

export ISAACLAB_PATH=/home/$USER/Desktop/booster_train/IsaacLab
alias isaaclab=/home/$USER/Desktop/booster_train/IsaacLab/isaaclab.sh

export RESOURCE_NAME="IsaacSim"
source /home/$USER/Desktop/booster_train/IsaacLab/_isaac_sim/setup_conda_env.sh
```

> **Atenção:** Se você renomear ou mover a pasta `IsaacLab`, atualize este arquivo.

---

## 4. Instalar os pacotes Python

Com o ambiente conda ativado (`conda activate isaaclab_env`):

```bash
# 1. Corrigir dependência legada do Isaac Lab
pip install --no-build-isolation flatdict==4.0.1

# 2. Instalar Isaac Lab e seus submódulos
cd ~/Desktop/booster_train/IsaacLab
pip install -e source/isaaclab
pip install -e source/isaaclab_assets
pip install -e source/isaaclab_rl
pip install -e source/isaaclab_tasks

# 3. Instalar os assets do Booster
cd ~/Desktop/booster_train
git clone https://github.com/BoosterRobotics/booster_assets.git  # se ainda não tiver
pip install -e booster_assets

# 4. Instalar este repositório
cd ~/Desktop/booster_train/booster_train
pip install -e source/booster_train
```

---

## 5. Verificar instalação

```bash
python -c "import isaaclab; print('isaaclab OK')"
python -c "import booster_assets; print('booster_assets OK')"
python -c "import booster_train; print('booster_train OK')"
```

Listar todos os ambientes registrados:

```bash
python scripts/list_envs.py
```

Você deve ver os ambientes do T1:
```
Booster-T1-Locomotion-Flat-v0
Booster-T1-Locomotion-Rough-v0
Booster-T1-Locomotion-v0-Play
Booster-T1-Dance-v0
...
```

---

## 6. Treinar o T1 andando

```bash
cd ~/Desktop/booster_train/booster_train

# Terreno plano (mais rápido para convergir, bom pra começar)
python scripts/rsl_rl/train.py --task Booster-T1-Locomotion-Flat-v0 --headless

# Terreno rugoso com curriculum (mais robusto, demora mais)
python scripts/rsl_rl/train.py --task Booster-T1-Locomotion-Rough-v0 --headless

# Especificar GPU
python scripts/rsl_rl/train.py --task Booster-T1-Locomotion-Flat-v0 --headless --device cuda:0
```

Os logs e checkpoints são salvos em:
```
logs/rsl_rl/t1_locomotion/<data_hora>/
```

---

## 7. Visualizar a política treinada

```bash
python scripts/rsl_rl/play.py \
    --task Booster-T1-Locomotion-v0-Play \
    --checkpoint logs/rsl_rl/t1_locomotion/<run>/model_<iter>.pt
```

---

## Tarefas disponíveis

| Task ID | Descrição |
|---------|-----------|
| `Booster-T1-Locomotion-Flat-v0` | Locomoção T1, terreno plano, sem estimador de estado |
| `Booster-T1-Locomotion-Rough-v0` | Locomoção T1, terreno rugoso + escadas, com curriculum |
| `Booster-T1-Locomotion-v0-Play` | Play/visualização (1 env, sem perturbações) |
| `Booster-T1-Dance-v0` | Dança T1 (motion tracking, requer NPZ) |
| `Booster-K1-MJ_Dance_004-v0` | Dança K1 (motion tracking) |

---

## Troubleshooting

**`No module named 'isaaclab'`**
→ O `setenv.sh` do conda está com caminho errado, ou o pacote não foi instalado.
Verifique o arquivo `activate.d/setenv.sh` e rode `pip install -e source/isaaclab` novamente.

**`Failed to build flatdict`**
→ Use `pip install --no-build-isolation flatdict==4.0.1` antes de instalar o `isaaclab`.

**`No module named 'booster_assets'`**
→ Rode `pip install -e ~/Desktop/booster_train/booster_assets`.

**`No module named 'pkg_resources'`**
→ Rode `pip install setuptools` e tente novamente.

**O conda não carrega as variáveis após renomear pastas**
→ Edite `~/miniconda3/envs/isaaclab_env/etc/conda/activate.d/setenv.sh` com o novo caminho e faça `conda deactivate && conda activate isaaclab_env`.
