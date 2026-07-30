# 🚀 aIRQFuzz (BLE) - QEMU Directed Firmware Protocol Fuzzer

An emulation-based, directed fuzzing framework that automatically discovers vulnerabilities deep into the wireless protocol implementation of bare-metal firmware. We evaluate **aIRQFuzz** on two distinct targets (BLE and Zigbee) to demonstrate both its effectiveness and its extensibility. aIRQFuzz opens possibilities for emulation-based and stateful fuzzing of complex wireless protocols.

As of today, 4 new CVEs in Nordic Zephyr stack have been assigned: *CVE-2025-12890, CVE-2025-65620, CVE-2025-70905* and *CVE-2025-65621*. 


<p align="center">
  <img src="figs/Overview_Update_page-0001.jpg" alt="aIRQFuzz Overview and Design" width="800">
</p>

---


**Table of Contents**

- [🚀 aIRQFuzz (BLE) - QEMU Directed Firmware Protocol Fuzzer](#-airqfuzz-ble---qemu-directed-firmware-protocol-fuzzer)
  - [📋 1. Software Environment](#-1-software-environment)
  - [⏩ 2. Initial Compilation](#-2-initial-compilation)
  - [🔀 3. Running Emulation Exploration](#-3-running-emulation-exploration)
    - [3.1 Target Firmware BLE](#31-target-firmware-ble)
    - [3.2 Target Config BLE](#32-target-config-ble)
    - [3.3 Target Patch BLE](#33-target-patch-ble)
  - [🧑‍💻 4. Input Runner](#-4-input-runner)
    - [4.1 Run single input without fuzzing engine](#41-run-single-input-without-fuzzing-engine)
    - [4.2 Run single input with mmio/ram access documented](#42-run-single-input-with-mmioram-access-documented)
    - [4.1 Run single input with fuzzing engine](#41-run-single-input-with-fuzzing-engine)
  - [📄 5. Running the Fuzzer](#-5-running-the-fuzzer)
    - [5.1 Customized U-fuzz docker image](#51-customized-u-fuzz-docker-image)
    - [5.2 Running Tutorial](#52-running-tutorial)
  - [🚨 6. Exploits](#-6-exploits)
    - [6.1.  Summary of potential Crashes:](#61--summary-of-potential-crashes)
      - [QPF effectiveness to find/replicate crashes](#qpf-effectiveness-to-findreplicate-crashes)
    - [6.2. Available Exploits](#62-available-exploits)
    - [6.3. Real board replication](#63-real-board-replication)
    - [6.4. Emulation replication](#64-emulation-replication)
    - [6.5. Auto Verification Potential PoC on Multiple Emulation Input](#65-auto-verification-potential-poc-on-multiple-emulation-input)
  - [⚙️ 7. PoC Script Auto-generator](#️-7-poc-script-auto-generator)
  - [⚖️ 8. Auto Weight Calculator](#️-8-auto-weight-calculator)
  - [Prerequisites](#prerequisites)
  - [Core Workflow Example](#core-workflow-example)
    - [Step 1: Import Symbols from an ELF file](#step-1-import-symbols-from-an-elf-file)
    - [For BLE](#for-ble)
    - [For Zigbee](#for-zigbee)
    - [Step 2: Search for Symbols and Generate Call Traces](#step-2-search-for-symbols-and-generate-call-traces)
    - [Step 3: Merge All Call Traces](#step-3-merge-all-call-traces)
    - [Step 4: Use the Generated Hooks](#step-4-use-the-generated-hooks)
  - [📝 9. Citing aIRQFuzz](#-9-citing-airqfuzz)

> [!NOTE]
> Should you encounter any technical difficulties reproducing our results, please do not hesitate to contact us at **zewena66@gmail.com**. We maintain a pre-configured virtual machine for testing and evaluation, which can be securely accessed via Tailscale upon request.

------

## 📋 1. Software Environment
* **OS:** Ubuntu 24.04 - We recommend using Ubuntu 24.04 to build and run the emualtion engine for aIRQFuzz. As for the fuzzing engine, we prepared a ready-to-run docker [container](#51-customised-u-fuzz-docker-image) which build on ubuntu-18.04.  Alternativelly, you can refer to [U-fuzz]([url](https://github.com/asset-group/U-Fuzz/blob/main/README.md#2--initial-compilation)) github repo for environment setup to ensure the correct OS environment.

## ⏩ 2. Initial Compilation 
Several requirements need to be installed before compiling the project. An automated script for Ubuntu 24.04 is provided on `requirements.sh`. To compile from source, simply run the following commands:
```bash
# Download the content from this github link:
https://anonymous.4open.science/r/AIRQFuzz_ZBE/

$ cd aIRQFuzz_ble

$ ./requirements.sh # Create a python virtual environment and Install all requirements to compile emulator from source 

$ cargo build # Compile all binaries. It may take around 15min. Go get a coffe!
```

## 🔀 3. Running Emulation Exploration
Before running the emulation engine three inputs need to be provided as follows:

1: Target Firmware Binary and Elf

2: Target configuration which specifies the memmory layout for the target firmware, the exithook function list and Interrupt Injection method.

3: Customized Patch for Target whcih contains the necessary patch for advancing the emulation to protocol code space.

After compiling the project with the correct software environment, please run the following command
```bash
$ cd qpfuzzer_ble

target fodler was specified in fuzz.sh
$ ./fuzz.sh
```
The emulation log would be saved in the target fodler under name log-fuzzing.txt
<!-- ## 3.1 Target Firmware BLE
[Target Fimware BLE](TODO)
## 3.2 Target Config BLE
## 3.3 Target Patch BLE -->
## 3.1 Target Firmware BLE
[Ble binary](./target-zephyr/firmware/)
[Ble elf](./target-zephyr/firmware/)
## 3.2 Target Config BLE
[Ble config](./target-zephyr/config_multi_version/)
## 3.3 Target Patch BLE
[Ble patch](./target-zephyr/hook_multi_version/)
[Ble patch for fuzzing](./target-zephyr/hook_multi_version/)

## 🧑‍💻 4. Input Runner
Once meaningful emulation session has been done. a corpus archive file will be saved at target_folder/runs directory. After un-tar it, individual file could be retrieved.

<!-- See help for details:
```
cargo run --bin hoedur-arm -- fuzz --help
``` -->

## 4.1 Run single input without fuzzing engine

First of all need to go the the qpfuzzer_ble directory by running the following cmd
```bash
$ cd qpfuzzer_ble
```
Then before running the single input with different ble firmware versions, several files need to be updated to make sure the emulation process is replicable

**Step1:**
*Update the run-inpuit.sh*
The file is located at (./qpfuzzer_ble).

1. update the hook file path to hook_without_fuzzer.rs with targeted version instead of using the hook.rs which is created for liveing fuzzing using socket.
   All hook files could be located at [Ble patch](./target-zephyr/hook_multi_version/)

**Step2:**
*Update the config.yml*
The file is located at the (./interval-500-fuzzed-clock-10t/config.yml)
Update the content based on the config file for different verison. 
All config file are loacted at [Ble config](./target-zephyr/config_multi_version/)

**Step3:**
*Copy the targeted Firmware*
The targeted firmware needs to be specified in the config file and the binary need to be copy and paste into the (./interval-500-fuzzed-clock-10t/).
All firmware could be located at [Ble binary](./target-zephyr/firmware/) and [Ble elf](./target-zephyr/firmware/).

**Step4:**
*Update the Emulation handling logic*
The emulator will handle the firmware with different verison slightly different. Since different version requires different targeted keywords etc. 
The file (qpfuzzer_ble/emulator/src/hooks/custom/common.rs) and (qpfuzze_4.1/emulator/src/lib.rs) needs to be updated for different targeted version. The reference code could be located at [common](./target-zephyr/common_multi_version/) and [lib](./target-zephyr/lib_multi_version/)

**Step5:**
*Update the Coverage handling logic*
The hardware.rs file (qpfuzzer_ble/modeling/src/hardware.rs) also need to be updated to make sure the emulation is replicable. The reference code could be located at [hardware](./target-zephyr/hardware_multi_version/)


**Step6:**
Then use the following cmd to run the targted input file for targeted version
```
$ ./run-input.sh <input.bin>
```
All input file could be loacted at [Individual input Ble](./target-zephyr/meaningful_input_multi_version/)

## 4.2 Run single input with mmio/ram access documented
After all update done previously, the following cmd could be used to run single input
```bash
$ cd qpfuzzer_ble

$ ./run-input-detail.sh <input.bin>
```
## 4.1 Run single input with fuzzing engine
To show the interception and live decoding of the fuzzer, besides all the steps we went through at 4.1. we only need to change the replace the hook file from (hook_without_fuzzer.rs) to (hook.rs).

Before running the input, the **U-fuzz** fuzzing engine need to be run by folloing the 5. tutorial.  

## 📄 5. Running the Fuzzer
## 5.1 Customized U-fuzz docker image
*Can pull from docker hub*
```
docker pull airqfuzz/u-fuzz-docker:aIRQFuzz
```
## 5.2 Running Tutorial
**Step1:**
*build the project (ble_realtime_fuzzer)*

```cmake
# Edit the CMakeLists.txt
$ Comments line:802, 811-815 
  (`set(ZIGBEE_SRC src/zigbee_realtime_fuzzer.cpp libs/shared_memory.c)`)
  (`add_executable(zigbee_realtime_fuzzer ${ZIGBEE_SRC} libs/profiling.c)`)
  (`target_link_libraries(zigbee_realtime_fuzzer PRIVATE ${MINIMAL_FUZZER_LIBS} viface)`)
  (`target_compile_options(zigbee_realtime_fuzzer PRIVATE -w -O0)`)
  (`target_compile_definitions(zigbee_realtime_fuzzer PRIVATE -DFUZZ_WIFI_AP)`)

$ Uncomments line: 806, 825-829 which were configured for Ble fuzzing
  (set(BLE_SRC src/ble_realtime_fuzzer.cpp libs/shared_memory.c))
  (add_executable(ble_realtime_fuzzer ${BLE_SRC} libs/profiling.c))
  (target_link_libraries(ble_realtime_fuzzer PRIVATE ${MINIMAL_FUZZER_LIBS} viface))
  (target_compile_options(ble_realtime_fuzzer PRIVATE -w -O0))
  (target_compile_definitions(ble_realtime_fuzzer PRIVATE -DFUZZ_WIFI_AP))

$ ./build.sh all

```
**Step2:**
*Update the fuzzing config if needed*
```bash
$ sudo nano /home/user/U-Fuzz/configs/ble_config.json
```
set enable_mutation == true and enable_optimization == true to enable stateful protocol aware fuzzing

**Step3:**
*Running the fuzzer*
```bash
$ cd /home/user/U-Fuzz
# Enable the mutation("enable_mutation": True) and optimization("enable_optimization": True) for the fuzzer by update the config
$ sudo nano configs/ble_config.json
# Once config is done, run the following cmd to start the fuzzing engine
$ sudo bin/ble_realtime_fuzzer
```
The fuzzing probability, max fuzzing time and max iteration could also be updated in the config file. More details could be found in [U-fuzz repo]([url](https://github.com/asset-group/U-Fuzz/))

**Step4:**
*Replay the emulation single input  at aIRQFuzz*
```bash
$ cd ~/qpfuzer_ble

# This step will loop the emulation input over and over again to allow the fuzzer to fuzz the communication process.
$ ./run-input-loop.sh <input.bin>
```
[Potential input](#43-emulation-input) were provided
**potential cmd:**
```bash
./run-input-loop.sh ./target-zephyr/meaningful_input_multi_version/v350/sm_pairing_req_good350_ss2.bin
```
 
For fuzzing the emulation, the fuzzing engine needs to be connected with the emulation engine to intercept the communication. [Ble patch for fuzzing](./target-zephyr/hook.rs) was required instead of 
[Ble patch](./target-zephyr/hook_without_fuzzer.rs).


## 🚨 6. Exploits
## 6.1.  Summary of potential Crashes:
To this day, aIRQFUZZ has found 22 potential crashes in the BLE implementation of Zephyr OS across multiple versions and 7 potential crashes in Zephyr/Nordic Zigbee implementation. 
### QPF effectiveness to find/replicate crashes

| Protocol | Fw. Version                  | Unique Crash | # Mutations | Potential Crash after multi-step filtering| Board Replication |
|----------|------------------------------|--------------|-------------|-----------------|-------------------|
| **BLE**  | V2.2.99                      | 78           | ≤ 3         | 11              | 2 (CVE-2020-10061, CVE-2020-10069) |
|          | V2.5.1                       | 2            | ≤ 3         | 2               | 0                 |
|          | V3.5.99                      | 5            | ≤ 3         | 1               | 1 (New:CVE-2025-12890)                 |
|          | V3.7.1                       | 18           | ≤ 2         | 6              | 1 (duplicate to V3.5) |
|          | V4.0.0                       | 13           | ≤ 3         | 2              | 1 (duplicate to V3.5) |
| **Total**| All versions                 | 116          | ≤ 3         | 22              | 3                 |
| **Zigbee** | Nordic V2.9.99 + Zephyr OS V3.7.9 | 27   | ≤ 5         | 7 (New: CVE-2025-65620, CVE-2025-70905)             | NA                |


## 6.2. Available Exploits
<details>
<summary><b>View BLE v2.2.99 Exploits</b></summary>

| Vulnerability Name | Exploit |
| --- | --- |
| t1_1_connect_ind | [t1_1_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_1_connect_ind.cpp) |
| t1_4_connect_ind | [t1_4_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_4_connect_ind.cpp) |
| t1_6_sent_exchange_mtu_request_client_rx_mtu_247 | [t1_6_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_6_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t1_9_sent_exchange_mtu_request_client_rx_mtu_247 | [t1_9_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_9_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t1_10_ll_version_ind | [t1_10_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_10_ll_version_ind.cpp) |
| t1_11_ll_version_ind | [t1_11_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_11_ll_version_ind.cpp) |
| t1_12_ll_feature_req | [t1_12_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_12_ll_feature_req.cpp) |
| t1_13_ll_feature_req | [t1_13_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_13_ll_feature_req.cpp) |
| t1_14_ll_length_req | [t1_14_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_14_ll_length_req.cpp) |
| t1_24_ll_feature_req | [t1_24_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_24_ll_feature_req.cpp) |
| t1_26_connect_ind | [t1_26_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_26_connect_ind.cpp) |
| t1_27_connect_ind | [t1_27_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_27_connect_ind.cpp) |
| t1_28_connect_ind | [t1_28_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_28_connect_ind.cpp) |
| t1_29_connect_ind | [t1_29_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t1_29_connect_ind.cpp) |
| t2_1_ll_version_ind | [t2_1_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_1_ll_version_ind.cpp) |
| t2_1_sent_exchange_mtu_request_client_rx_mtu_247 | [t2_1_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_1_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_4_sent_exchange_mtu_request_client_rx_mtu_247 | [t2_4_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_4_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_5_sent_exchange_mtu_request_client_rx_mtu_247 | [t2_5_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_5_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_6_sent_exchange_mtu_request_client_rx_mtu_247 | [t2_6_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_6_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_7_ll_length_req | [t2_7_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_7_ll_length_req.cpp) |
| t2_7_sent_exchange_mtu_request_client_rx_mtu_247 | [t2_7_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_7_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_8_ll_length_req | [t2_8_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_8_ll_length_req.cpp) |
| t2_8_ll_version_ind | [t2_8_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_8_ll_version_ind.cpp) |
| t2_9_ll_length_req | [t2_9_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_9_ll_length_req.cpp) |
| t2_9_ll_version_ind | [t2_9_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_9_ll_version_ind.cpp) |
| t2_9_sent_exchange_mtu_request_client_rx_mtu_247 | [t2_9_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_9_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_10_ll_length_req | [t2_10_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_10_ll_length_req.cpp) |
| t2_10_ll_version_ind | [t2_10_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_10_ll_version_ind.cpp) |
| t2_11_ll_feature_req | [t2_11_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_11_ll_feature_req.cpp) |
| t2_11_ll_length_req | [t2_11_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_11_ll_length_req.cpp) |
| t2_11_ll_version_ind | [t2_11_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_11_ll_version_ind.cpp) |
| t2_12_ll_feature_req | [t2_12_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_12_ll_feature_req.cpp) |
| t2_12_ll_length_req | [t2_12_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_12_ll_length_req.cpp) |
| t2_13_ll_feature_req | [t2_13_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_13_ll_feature_req.cpp) |
| t2_13_sent_exchange_mtu_request_client_rx_mtu_247 | [t2_13_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_13_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_14_ll_feature_req | [t2_14_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_14_ll_feature_req.cpp) |
| t2_14_ll_length_req | [t2_14_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_14_ll_length_req.cpp) |
| t2_14_sent_exchange_mtu_request_client_rx_mtu_247 | [t2_14_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_14_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_24_ll_feature_req | [t2_24_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t2_24_ll_feature_req.cpp) |
| t3_1_ll_version_ind | [t3_1_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_1_ll_version_ind.cpp) |
| t3_1_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_1_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_1_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_2_ll_version_ind | [t3_2_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_2_ll_version_ind.cpp) |
| t3_3_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_3_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_3_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_4_ll_version_ind | [t3_4_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_4_ll_version_ind.cpp) |
| t3_4_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_4_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_4_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_4_sent_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk | [t3_4_sent_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_4_sent_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk.cpp) |
| t3_5_ll_length_req | [t3_5_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_5_ll_length_req.cpp) |
| t3_5_ll_version_ind | [t3_5_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_5_ll_version_ind.cpp) |
| t3_5_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_5_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_5_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_6_ll_length_req | [t3_6_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_6_ll_length_req.cpp) |
| t3_6_ll_version_ind | [t3_6_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_6_ll_version_ind.cpp) |
| t3_6_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_6_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_6_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_7_ll_length_req | [t3_7_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_7_ll_length_req.cpp) |
| t3_7_ll_version_ind | [t3_7_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_7_ll_version_ind.cpp) |
| t3_7_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_7_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_7_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_8_ll_length_req | [t3_8_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_8_ll_length_req.cpp) |
| t3_8_ll_version_ind | [t3_8_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_8_ll_version_ind.cpp) |
| t3_8_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_8_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_8_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_8_sent_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk | [t3_8_sent_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_8_sent_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk.cpp) |
| t3_9_ll_length_req | [t3_9_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_9_ll_length_req.cpp) |
| t3_9_ll_version_ind | [t3_9_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_9_ll_version_ind.cpp) |
| t3_9_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_9_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_9_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_10_ll_length_req | [t3_10_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_10_ll_length_req.cpp) |
| t3_10_ll_version_ind | [t3_10_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_10_ll_version_ind.cpp) |
| t3_11_ll_length_req | [t3_11_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_11_ll_length_req.cpp) |
| t3_11_ll_version_ind | [t3_11_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_11_ll_version_ind.cpp) |
| t3_11_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_11_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_11_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_12_ll_length_req | [t3_12_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_12_ll_length_req.cpp) |
| t3_13_ll_version_ind | [t3_13_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_13_ll_version_ind.cpp) |
| t3_13_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_13_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_13_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_14_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_14_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_14_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_15_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_15_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_15_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_18_sent_exchange_mtu_request_client_rx_mtu_247 | [t3_18_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_18_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t3_25_ll_length_req | [t3_25_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_25_ll_length_req.cpp) |
| t3_27_ll_version_ind | [t3_27_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v220/t3_27_ll_version_ind.cpp) |

</details>

<details>
<summary><b>View BLE v2.5.0 Exploits</b></summary>

| Vulnerability Name | Exploit |
| --- | --- |
| t1_2_connect_ind | [t1_2_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v250/t1_2_connect_ind.cpp) |
| t1_3_connect_ind | [t1_3_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v250/t1_3_connect_ind.cpp) |

</details>

<details>
<summary><b>View BLE v3.5.0 Exploits</b></summary>

| Vulnerability Name | Exploit |
| --- | --- |
| t1_8_ll_version_ind | [t1_8_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t1_8_ll_version_ind.cpp) |
| t1_11_ll_length_req | [t1_11_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t1_11_ll_length_req.cpp) |
| t2_5_rcvd_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk | [t2_5_rcvd_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t2_5_rcvd_pairing_request_authreq_bonding_secureconnection__initiator_keys_ltk_irk_csrk__responder_keys_ltk_irk_csrk.cpp) |
| t2_6_ll_version_ind | [t2_6_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t2_6_ll_version_ind.cpp) |
| t2_8_ll_length_req | [t2_8_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t2_8_ll_length_req.cpp) |
| t2_8_ll_version_ind | [t2_8_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t2_8_ll_version_ind.cpp) |
| t2_9_ll_length_req | [t2_9_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t2_9_ll_length_req.cpp) |
| t2_11_ll_length_req | [t2_11_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t2_11_ll_length_req.cpp) |
| t3_5_ll_version_ind | [t3_5_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t3_5_ll_version_ind.cpp) |
| t3_6_ll_version_ind | [t3_6_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t3_6_ll_version_ind.cpp) |
| t3_8_ll_version_ind | [t3_8_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v350/t3_8_ll_version_ind.cpp) |

</details>

<details>
<summary><b>View BLE v3.7.1 Exploits</b></summary>

| Vulnerability Name | Exploit |
| --- | --- |
| t1_2_connect_ind | [t1_2_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t1_2_connect_ind.cpp) |
| t1_2_ll_feature_req | [t1_2_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t1_2_ll_feature_req.cpp) |
| t1_3_ll_feature_req | [t1_3_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t1_3_ll_feature_req.cpp) |
| t1_4_connect_ind | [t1_4_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t1_4_connect_ind.cpp) |
| t1_5_connect_ind | [t1_5_connect_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t1_5_connect_ind.cpp) |
| t1_5_ll_feature_req | [t1_5_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t1_5_ll_feature_req.cpp) |
| t1_33_ll_feature_req | [t1_33_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t1_33_ll_feature_req.cpp) |
| t1_36_ll_feature_req | [t1_36_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t1_36_ll_feature_req.cpp) |
| t2_2_ll_feature_req | [t2_2_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_2_ll_feature_req.cpp) |
| t2_3_ll_feature_req | [t2_3_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_3_ll_feature_req.cpp) |
| t2_5_ll_feature_req | [t2_5_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_5_ll_feature_req.cpp) |
| t2_8_ll_length_req | [t2_8_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_8_ll_length_req.cpp) |
| t2_9_ll_feature_req | [t2_9_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_9_ll_feature_req.cpp) |
| t2_32_ll_feature_req | [t2_32_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_32_ll_feature_req.cpp) |
| t2_33_ll_feature_req | [t2_33_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_33_ll_feature_req.cpp) |
| t2_34_ll_feature_req | [t2_34_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_34_ll_feature_req.cpp) |
| t2_35_ll_feature_req | [t2_35_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v371/t2_35_ll_feature_req.cpp) |

</details>

<details>
<summary><b>View BLE v4.1.0 Exploits</b></summary>

| Vulnerability Name | Exploit |
| --- | --- |
| t1_10_ll_length_req | [t1_10_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t1_10_ll_length_req.cpp) |
| t1_11_ll_length_req | [t1_11_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t1_11_ll_length_req.cpp) |
| t1_12_ll_feature_req | [t1_12_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t1_12_ll_feature_req.cpp) |
| t1_14_ll_feature_req | [t1_14_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t1_14_ll_feature_req.cpp) |
| t1_14_sent_exchange_mtu_request_client_rx_mtu_247 | [t1_14_sent_exchange_mtu_request_client_rx_mtu_247.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t1_14_sent_exchange_mtu_request_client_rx_mtu_247.cpp) |
| t2_3_ll_version_ind | [t2_3_ll_version_ind.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t2_3_ll_version_ind.cpp) |
| t2_6_ll_length_req | [t2_6_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t2_6_ll_length_req.cpp) |
| t2_9_ll_length_req | [t2_9_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t2_9_ll_length_req.cpp) |
| t2_10_ll_length_req | [t2_10_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t2_10_ll_length_req.cpp) |
| t2_11_ll_length_req | [t2_11_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t2_11_ll_length_req.cpp) |
| t2_12_ll_feature_req | [t2_12_ll_feature_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t2_12_ll_feature_req.cpp) |
| t3_4_ll_length_req | [t3_4_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t3_4_ll_length_req.cpp) |
| t3_10_ll_length_req | [t3_10_ll_length_req.cpp](./target-zephyr/ble_exploits_multi_version/ble_v410/t3_10_ll_length_req.cpp) |

</details>

## 6.3. Real board replication
Our group used nrf52840DK board to verify the potential crash on the real board.
To launch such attack, please follow the attack tutorial that vakt-ble provided in section [4.1 Launching Sweyntooth Attacks](https://github.com/asset-group/vakt-ble-defender?tab=readme-ov-file#41-launching-sweyntooth-attacks)


## 6.4. Emulation replication
Emulation replication requires the auto-generated [PoC scripts](#62-available-exploits) running by the fuzzing engine to replay the crash sequence. Both the enable_mutation and enable_optimization need to be set to false to eliminate the normal mutation operation.
The detailed emulation replication [tutorial](./tutorial/emulation_replication_tutorial.html) was provided.
The replay result should looks like the following 
<p align="center">
  <img src="figs/crash_in_fuzzing.png" alt="crash fuzing">
</p>
<!-- Add a file for tutorial -->

## 6.5. Auto Verification Potential PoC on Multiple Emulation Input
As mentioned in the paper, one of our crash filtering step is to verify the potential crash on multiple emulation input. We created a [script](./QPFuzzer/scripts/auto_test_crash.sh) for this automatic testing.

```bash
# After the emulation engine is configured for specific target, the fuzzing engine also need to be configured to target the exploit folder for that target and the mutation and optimization flag also need to be enabled.

Once set up, run the following cmd to auto verify the potential crash

Enter screen session to control running u-fuzz docker
$ screen -r u-fuzz

$ sudo bin/ble_realtime_fuzzer --exploit=<Specific exploit from exploit list>

Exit the screen session
$ ctrl + A + D 

Run the auto test script 
$ cd airqfuzz_ble
$ ./script/auto_test_crash.sh
```
Expected result should looks like
<p align="center">
  <img src="figs/auto_verification.png" alt="Auto Verification">
</p>

## ⚙️ 7. PoC Script Auto-generator
This script analyzes `.pcapng` log files from a fuzzing session. It identifies crash-causing packet sequences, generates a CSV summary, and creates C++ scripts to reproduce potential exploits using U-Fuzz fuzzing framework.

```bash
pip install pandas typer pcapng numpy tqdm
```

Analyze a log for the `zigbee` or `ble` protocol and save the report to a custom file named `results.csv`.

```bash
# For BLE
python analyse_log.py zigbee_run.pcapng --output-file results.csv --protocol-name ble

# For Zigbee
python analyse_log.py zigbee_run.pcapng --output-file results.csv --protocol-name zigbee
```

The script performs two main actions:

1.  **Generates a CSV Report**: It creates a `.csv` file (e.g., `capture.csv`) summarizing each fuzzing iteration, sorted to highlight the most promising results (low number of fuzzed packets before a crash). The report includes columns like `Iteration`, `Crashes`, `Fuzz Count`, and `Fuzz-End Distance`.

2.  **Generates Trial Scripts**: For each identified crash, it generates a C++ script under folder `exploits` (e.g., `exploits/zigbee/t1_25_beaconrsp.cpp`). These scripts are designed to reproduce the exact sequence of packets that caused the crash, and should be moved to the `modules/exploits/zigbee` folder of the fuzzer engine. 


## ⚖️ 8. Auto Weight Calculator

## Prerequisites

1. **Python Dependencies**: Install the required packages.

   ```bash
   pip install typer chromadb pandas numpy matplotlib binaryninja cmsis-svd tqdm PyYAML
   ```

   *Note: A valid Binary Ninja license is required for the `binaryninja` package.*

2. **Embedding Service**: The tool requires an [infinity_emb](https://github.com/michaelfeil/infinity) embedding model server to be running. Set its URL via an environment variable.

   ```bash
   export EMBEDDINGS_URL="http://127.0.0.1:7997"
   ```

## Core Workflow Example

This example demonstrates the end-to-end process of analyzing an ELF file to generate Rust hook files for a fuzzer.

### Step 1: Import Symbols from an ELF file

First, ingest the function symbols from your compiled firmware into the vector database. This needs to be done only once per file.
> This reads symbols from `firmware.elf`, generates embeddings, and saves them in the `firmware_db/` directory.

### For BLE
```bash
cd scripts/
python3 symbol-analyzer.py import-elf --elf-path ../target-zephyr/zephyr.elf
```

### For Zigbee
```bash
cd scripts/
python3 symbol-analyzer.py import-elf --elf-path ../target-zigbee/zigbee.elf
```

### Step 2: Search for Symbols and Generate Call Traces

Use natural language queries to find relevant functions and automatically generate their call traces. The tool uses Binary Ninja in the background to analyze the callers for each found symbol.


```bash
# For BLE
./symbol-analyzer.py search-calltrace "smp pairing" --n 20
./symbol-analyzer.py search-calltrace "att message" --n 20
./symbol-analyzer.py search-calltrace "gatt message" --n 20
./symbol-analyzer.py search-calltrace "l2cap message" --n 20
./symbol-analyzer.py search-calltrace "link layer message" --n 20
./symbol-analyzer.py search-calltrace "bluetooth advertisement indication" --n 20
./symbol-analyzer.py search-calltrace "bluetooth scan response" --n 20
./symbol-analyzer.py search-calltrace "bluetooth gap profile" --n 20
./symbol-analyzer.py search-calltrace "bluetooth radio phy transmit" --n 20
./symbol-analyzer.py search-calltrace "bluetooth radio phy receive" --n 20
./symbol-analyzer.py search-calltrace "bluetooth radio interrupt" --n 20
./symbol-analyzer.py search-calltrace "bluetooth radio handling" --n 20

# For Zigbee
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "beacon request"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "scan request"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "radio tx init"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "radio rx init"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "radio transmission"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "radio reception"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "zigbee state machine"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "zigbee protocol message"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "zigbee network message"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "zigbee mac message"
./symbol-analyzer.py search-calltrace --n=20 --folder=calltraces-zigbee --file="zigbee.elf" "ack rx"
```


> For each command, this creates `.json` and `.yaml` backtrace files inside the `calltraces/` directory.

### Step 3: Merge All Call Traces

Combine all the individual JSON backtrace files generated in the previous step into a single file. This command also generates Rust hook files for instrumentation.

```bash
# For BLE
python symbol-analyzer.py merge-calltraces --calltrace-folder calltraces

# For Zigbee
python symbol-analyzer.py merge-calltraces --calltrace-folder calltraces-zigbee
```

> This creates:
>
> - `merged_calltrace.json`: A combined JSON of all unique functions and their weights.
> - `hook-traces.rs`: A Rust file to log when a function is executed.
> - `hook-weights.rs`: A Rust file to assign weights to basic blocks for guided fuzzing.

### Step 4: Use the Generated Hooks

Finally, copy the generated Rust files into your target project (e.g., a fuzzer's hooks directory).

```bash
# For BLE
cp hook-weights.rs hook-traces.rs ../target-zephyr

# For Zigbee
cp hook-weights.rs hook-traces.rs ../target-zigbee


# Start the fuzzer with the weights
cargo run --release --bin hoedur-arm -- \
    --config target-zephyr/config.yml \
    --hook hook.rs \
    --hook hook-weights.rs \
    --debug \
    fuzz \
    --statistics \
    --archive-dir target-zephyr/runs
```

> You can now run the fuzzer with these new hooks to guide its execution based on your semantic searches. 


## 📝 9. Citing aIRQFuzz

```bibtex
@article{airqfuzz,
  title={aIRQFuzz: QEMU Directed Firmware Protocol Fuzzer},
  author={Zewen Shang, Matheus E. Garbelini, Sudipta Chattopadhyay},
  journal={To Be Added},
  year={2026}
}
```


<!-- 
Todo:
1. finished the fuzzing docker (separete ble and zigbee)
2. Update the ## 5.2 Running Tutorial
3. output the docker
4. realboard replication script
6. Finished the PoC generator part
7. Make sure the anonymouse
 -->
