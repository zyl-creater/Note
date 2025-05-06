

```shell
console:/ # tee_provision -h                                               
Amlogic Provision Tool v4.4.0
Usage: tee_provision [-i <file>] [-n <name>] [-t <type>] [-e <nonce str>] [-u <ta uuid>] [-s <encrypted keybox file>] [-q] [-l] [-c] [-p] [-a] [-d] [-o] [-v] [-h]

  -i, --in            Input data file
  -n, --name          Specify Efuse Object name
  -t, --type          Specify key type
  -e, --nonce         Nonce string for calculating dac
  -q, --query         Query key provisioned or not
  -d, --delete        Delete provisioned key
  -l, --list          List all key types
  -c, --checksum      Show provisioned raw key's sha256
  -p, --pfid          Get provision field id
  -a, --dac           Get device authentication code
  -u, --uuid          TA UUID for customer-defined key
  -s, --sha256        Calc raw key's sha256
  -o, --object        List all Efuse Object name
  -v, --version       Show tool version
  -h, --help          Show this help
```

### **工具概述**​

- ​**名称**: `tee_provision`（Amlogic Provision Tool）
- ​**版本**: v4.4.0
- ​**用途**: 用于在基于 Amlogic 芯片的设备中管理 TEE（可信执行环境）的密钥、证书等安全数据，支持密钥注入、查询、删除、校验等操作，通常用于设备生产或安全配置阶段。

---

### ​**参数详解**​

#### ​**1. 核心操作选项**​

|参数|全称|用途|示例|
|---|---|---|---|
|`-i`|`--in`|指定输入文件（需写入的密钥或数据文件）|`tee_provision -i key.bin -n aes_key`|
|`-n`|`--name`|指定 Efuse 对象名称（即写入目标的安全存储区域名称）|`-n secure_boot_key`|
|`-t`|`--type`|指定密钥类型（如对称密钥、非对称密钥等，需参考文档）|`-t AES256`|
|`-q`|`--query`|查询某个密钥是否已配置|`tee_provision -q -n root_key`|
|`-d`|`--delete`|删除已配置的密钥|`tee_provision -d -n drm_key`|
|`-l`|`--list`|列出所有支持的密钥类型|`tee_provision -l`|
|`-o`|`--object`|列出所有 Efuse 对象名称（可写入的安全存储区域）|`tee_provision -o`|

---

#### ​**2. 安全校验与工具功能**​

|参数|全称|用途|示例|
|---|---|---|---|
|`-c`|`--checksum`|显示已配置密钥的 SHA256 校验和|`tee_provision -c -n device_cert`|
|`-s`|`--sha256`|计算输入文件的 SHA256 校验和（用于验证原始数据）|`tee_provision -s keybox.enc`|
|`-a`|`--dac`|生成设备认证码（Device Authentication Code）|`tee_provision -a -n device_id`|
|`-e`|`--nonce`|指定随机字符串（Nonce），用于生成 DAC|`tee_provision -a -e "RANDOM_STR"`|
|`-p`|`--pfid`|获取设备的安全存储区域 ID（Provision Field ID）|`tee_provision -p`|
|`-u`|`--uuid`|指定客户自定义密钥的 TA（Trusted Application）UUID|`-u 12345678-...`|

---

#### ​**3. 辅助选项**​

|参数|全称|用途|
|---|---|---|
|`-v`|`--version`|显示工具版本|
|`-h`|`--help`|显示帮助信息|

---

### ​**典型使用场景**​

#### ​**场景 1：注入密钥**​

bash

复制

```bash
# 将文件 aes_key.bin 注入到名为 "secure_aes" 的 Efuse 区域，类型为 AES-256
tee_provision -i aes_key.bin -n secure_aes -t AES256
```

#### ​**场景 2：查询密钥状态**​

bash

复制

```bash
# 检查名为 "drm_cert" 的证书是否已配置
tee_provision -q -n drm_cert
```

#### ​**场景 3：生成设备认证码（DAC）​**​

bash

复制

```bash
# 使用随机字符串 "S3CR3T" 生成设备的认证码
tee_provision -a -n device_id -e "S3CR3T"
```

#### ​**场景 4：删除密钥**​

bash

复制

```bash
# 删除名为 "temp_key" 的临时密钥
tee_provision -d -n temp_key
```

---

### ​**注意事项**​

1. ​**权限要求**​：此工具通常需要 `root` 权限或与 TEE 安全服务交互的权限。
2. ​**生产环境操作**​：密钥注入和删除操作可能直接影响设备安全性，需在受控环境中进行。
3. ​**参数依赖**​：
    - 使用 `-i`（输入文件）时需配合 `-n` 和 `-t`。
    - 生成 DAC（`-a`）可能需要 `-e`（随机字符串）和 `-n`（设备 ID）。
4. ​**芯片兼容性**​：不同 Amlogic 芯片（如 S905、A311D）支持的密钥类型和 Efuse 对象可能不同，需参考硬件文档。
5. ​**数据安全**​：输入文件（如密钥）建议通过安全通道传输，避免泄露。

---

### ​**扩展参考**​

- ​**Amlogic TEE 文档**: 需查阅芯片厂商提供的《Secure Boot Guide》或《Key Provisioning Manual》。
- ​**Efuse 对象列表**: 运行 `tee_provision -o` 可查看当前设备支持的存储区域名称。
- ​**密钥类型列表**: 运行 `tee_provision -l` 查看支持的密钥类型（如 AES128、RSA2048 等）。

如果有具体芯片型号或使用场景，可进一步提供更针对性的操作流程。