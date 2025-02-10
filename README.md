# Tutorial prático sobre LVM no Debian 12

## Conceitos principais do LVM
LVM (Logical Volume Manager) é um sistema de gerenciamento de volumes lógicos que oferece flexibilidade na administração de armazenamento. Ele é composto por:

- **PV (Physical Volume)**: Dispositivos físicos, como discos ou partições.
- **VG (Volume Group)**: Agrupamento de um ou mais PVs, criando um pool de armazenamento.
- **LV (Logical Volume)**: Partições lógicas criadas dentro de um VG, similares a partições tradicionais.

## 1. Configurando LVM com 4 discos de 1TB
### Passo 1: Instalando o LVM
```bash
apt update && apt install lvm2 -y
```

### Passo 2: Criando os Physical Volumes (PVs)
```bash
pvcreate /dev/sd[b-e]
```

### Passo 3: Criando um Volume Group (VG)
```bash
vgcreate vg_data /dev/sd[b-e]
```

### Passo 4: Criando um Logical Volume (LV)
```bash
lvcreate -L 3.5T -n lv_storage vg_data
```

### Passo 5: Formatando e montando
```bash
mkfs.ext4 /dev/vg_data/lv_storage
mkdir /mnt/storage
mount /dev/vg_data/lv_storage /mnt/storage
```
Para montagem persistente, adicione ao `/etc/fstab`:
```bash
/dev/vg_data/lv_storage /mnt/storage ext4 defaults 0 2
```

## 2. Simulando a perda de um disco e a troca por outro
### Passo 1: Identificar o disco defeituoso
```bash
pvs
vgs
lvs
```
Se `/dev/sdb` falhou, remova-o do VG:
```bash
vgreduce --removemissing --force vg_data
```

### Passo 2: Substituir por um novo disco de 1TB
```bash
pvcreate /dev/sdb
vgextend vg_data /dev/sdb
```
Se um disco de 1TB não estiver disponível, utilize um de 2TB:
```bash
pvcreate /dev/sdf
vgextend vg_data /dev/sdf
```

### Passo 3: Ajustar o armazenamento caso o novo disco seja de 2TB
Se o disco de substituição for maior, podemos realocar espaço:
```bash
lvextend -l +100%FREE /dev/vg_data/lv_storage
resize2fs /dev/vg_data/lv_storage
```

Agora, seu volume foi restaurado com sucesso, garantindo a continuidade do armazenamento.
