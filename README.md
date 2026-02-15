# CMake-SPL-Embedded (C8T6) Template

这是一个 STM32F103C8T6（C8T6）工程模板，基于 **SPL（标准外设库）**，使用 **CMake** 构建，适合快速启动 STM32F103C8T6 嵌入式项目。

----

## 目录结构

```
template/
├─ core/               # 系统初始化（startup 文件、system_stm32f10x.c/h）
├─ spl/                # SPL 标准外设库
├─ user/               # 用户逻辑 (main.c 等)
├─ toolchain.cmake     # 交叉编译工具链配置
├─ *.ld                # MCU 链接脚本
└─ CMakeLists.txt      # CMake 项目配置
```

- **`core/`**：存放系统初始化、启动文件
- **`spl/`**：官方 SPL 库文件
- **`user/`**：用户应用逻辑及板级支持包（BSP）
- **`toolchain.cmake`**：CMake 交叉编译工具链文件
- **`.ld`**：MCU 链接脚本
- **`CMakeLists.txt`**：CMake 构建脚本  

----

## 前置要求

在开始使用本工程之前，请确保已安装以下工具：

- **ARM GCC 工具链**  
  用于编译 STM32 固件，确保已添加到系统 PATH。

- **OpenOCD**  
  用于调试和烧录 MCU，可通过 SWD/JTAG 接口连接开发板。

- **Zadig (Windows)**  
  安装 ST-Link 驱动，确保 USB 调试和烧录正常工作。

- **CLion IDE (推荐)**  
  支持 CMake 项目，适合嵌入式开发，也可使用其他 CMake 支持的 IDE。

> **推荐使用 CLion 打开本工程**  
>
> CLion 对 CMake 项目支持良好，方便进行：
>
> - **自动依赖管理**
> - **代码补全与导航**
> - **编译和调试**
> - **与 OpenOCD 配合调试**

----

## 环境配置

本节介绍如何在 Windows 系统下配置本工程的必要开发环境。

### 1. 安装 ARM 工具链

推荐使用 **STM32CubeCLT** 或官方 ARM GCC 工具链。  
确保以下可执行文件存在并可在命令行调用：

- `arm-none-eabi-gcc`
- `arm-none-eabi-objcopy`

**下载地址**
- [STM32CubeCLT 官方下载页面](https://www.st.com/en/development-tools/stm32cubeclt.html)
- 或直接安装 [ARM GNU Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm)

**安装完成后验证**

```bash
arm-none-eabi-gcc --version
arm-none-eabi-objcopy --version
```

### 2. 安装 OpenOCD

OpenOCD 用于调试和烧录 MCU，通过 SWD/JTAG 与开发板连接。
可以下载官方 Windows 版本，或使用 MSYS2 等方式安装。

**下载地址**

- [OpenOCD 官方网站](http://openocd.org/)
- Windows 可使用预编译版本，例如：[Shumatech OpenOCD Windows Release](https://github.com/gnu-mcu-eclipse/openocd/releases)

> **注意**：在 Windows 上需要将 OpenOCD 安装路径加入系统 PATH 环境变量。


**安装完成后验证**

```bash
openocd --version
```


### 3.Zadig 驱动

用于给 ST-Link / PWLink 安装 WinUSB 驱动，确保 USB 调试和烧录正常工作。

**操作步骤 / Steps:**

1. 从 [Zadig 官方网站](https://zadig.akeo.ie/) 下载并并运行 `zadig.exe`

2. 将 C8T6 开发板连接到电脑，打开 Zadig

3. 在 Zadig 中选择你的设备：

- 对 ST-Link V2，一般显示为 `STM32 STLink` 或 `STM32 Bootloader`
- 对 PW-Link，一般显示为 `DAPLink CMSIS-DAP` 或 `MBED Interface`

4. 选择驱动为 WinUSB
5. 点击 `Install Driver` 或 `Replace Driver`

驱动安装完成后，即可通过 OpenOCD 访问开发板进行烧录和调试。

----

## CLion 项目配置

推荐使用 **CLion IDE** 打开本工程，CLion 对 CMake 项目支持良好，便于后续开发与调试。

### 1. 配置 CLion 工具链 (只需一次)

1. 打开 CLion，点击 **File → Settings → Build, Execution, Deployment → CMake**

2. 点击 **Toolchains**，添加一个新的 Toolchain

3. 完善 Toolchain 信息：

   - **名称**：自取，例如 **ARM**
   - **构建工具**：保持默认即可
   - **C 编译器**：填写 `arm-none-eabi-gcc`  
   - **C++ 编译器**：填写 `arm-none-eabi-g++`  
   - **调试器**：选择 **Custom GDB executable**，填写 `arm-none-eabi-gdb`

4. 点击 **Apply → OK** 保存配置


> 该工具链设置仅需进行一次，保存后即可永久使用


### 2. 克隆本仓库并使用 CLion 打开

1. 在本地终端或命令行输入以下命令，将仓库克隆到本地：

```bash
git clone https://github.com/NKanoya/CMakeSPLTemplate.git
cd CMakeSPLTemplate
```

打开 CLion，点击 Open，选择本项目根目录以打开工程

### 3. 配置打开的 CLion 项目

在弹出的 **“打开项目向导”** 窗口中，按照以下步骤配置：

- **名称 / Name" 和 "构建类型 / Build Type**：自定义或保持默认（推荐使用默认 **Debug**）

- **工具链 / Toolchain**：选择步骤 1 中新建的工具链，例如 **ARM**  
  - 工具链只需创建一次，后续直接选择即可

- **生成器 / Generator**：保持默认

- **CMake 选项 / CMake Options**：填入以下内容  

    ```text
    -DCMAKE_TOOLCHAIN_FILE=arm-none-eabi-toolchain.cmake
    ```

点击 确定 保存配置。

完成后，CLion 会正式打开项目并自动加载 CMake。

如果 CMake 输出显示 `[已完成]` / `[Finished]`，则表示项目配置成功。

### 4. 更改项目名称 + 构建

#### 4. 更改项目名称并构建

1. 在 CLion 左上角的项目路径视图中，打开根目录下的 `CMakeLists.txt`

2. 定位到第十行（或 `CUSTOMIZED_PROJECT_NAME` 所在行）：

    ```cmake
    set(CUSTOMIZED_PROJECT_NAME stm32_template)
    ```

    - `stm32_template` 是当前项目名，可以修改为其它名称。

    - 构建生成的文件名将与项目名保持一致。

3. 保存修改后，在 CLion 右上角点击 "**Build / 构建**" 按钮启动构建。

4. 构建完成后，可在 **`cmake-build-xxx/`** 目录中找到生成的 `.elf` 文件。

到此，CLion 已经可以通过 CMake 完成目标文件的构建。

### 5. 一键烧录配置

构建完成的 .elf 文件可以借助 OpenOCD 烧录到开发板中。CLion 的 "运行" 按钮默认执行 "构建 + 本地运行构建文件"，而本地无法直接执行 .elf 文件。因此，我们需要修改运行配置，实现 "构建 + 烧录到单片机" 的一键操作。

1. 点击 CLion 左上角菜单键（图标右侧的“菜单条”），依次选择 "运行 → 编辑配置"。

2. 在 "**可执行文件**" 下拉菜单中选择 "自定义"，填入 OpenOCD 可执行文件路径。

    - 可执行文件位于 OpenOCD 安装目录下 `\bin\openocd.exe` 内，例如 `D:/Program/OpenOCD/openocd.exe`。

3. 填写 "**程序实参**" 

    - 如果使用 PWLink，可以填入：

      ```bash
      -f
      interface/cmsis-dap.cfg
      -f
      target/stm32f1x.cfg
      -c
      "program $env(BUILD_DIR_NAME)/stm32_template.elf verify reset exit"
      ```

    - 如果使用 STLink，可以填入：

      ```bash
      -f
      interface/stlink.cfg
      -f
      target/stm32f1x.cfg
      -c
      "program $env(BUILD_DIR_NAME)/stm32_template.elf verify reset exit"
      ```

4. "**工作目录**" 填入 `$ProjectFileDir$` 。

5. "**环境变量**" 填入

    ```text
    BUILD_DIR_NAME=cmake-build-debug
    ```

    - `cmake-build-debug` 是输出目录，如果输出目录是别的名称，需要修改。

5. 点击下方 "**运行 / Run**"，程序应能重新构建，并烧录到单片机中。

点击 "运行" 按钮后，如果下方运行窗口输出内容最后部分包含:

```
** Programming Finished **
** Verify Started **
** Verified OK **
** Resetting Target **
```

并返回 `0`，则表明烧录成功。

----

## 开发流程

配置完成后，可以在 CLion 中基于 CMake 进行开发、构建和烧录。

- 点击 CLion 左上角的 "构建" 按钮，或使用菜单 "构建 → 构建项目"：CMake 会根据当前构建类型（Debug/Release）生成对应的 ELF 文件，例如 cmake-build-debug/stm32_template.elf。

- 配置好 OpenOCD 后，点击 CLion 的 "运行" 按钮。CLion 会自动构建项目，并自动调用 OpenOCD 将 ELF 文件烧录到单片机

  确认控制台输出显示：

  ```
  ** Programming Finished **
  ** Verify Started **
  ** Verified OK **
  ** Resetting Target **
  ```

  则表示烧录成功。

- 如果在 CLion 中选择不同构建类型（Debug / Release），输出目录可能对应发生变动。

  如果更改后要使用 "运行" 按钮构建并烧录，注意修改运行配置中的环境变量 `BUILD_DIR_NAME=<新输出目录名>` ，以确保烧录的 ELF 文件路径正确。

----

## 参考资料

- OpenOCD 官方文档：http://openocd.org/doc


