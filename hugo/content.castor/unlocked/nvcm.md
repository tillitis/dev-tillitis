---
title: Program NVCM
weight: 2
---

# Program the NVCM

Programming the NVCM (Non-Volatile Configuration Memory) makes it
possible to securely write the bitstream with its secrets and make it
non-readable.

Programming the NVCM consists of these steps:

1. Download tool.
2. Program the NVCM.
3. Remove traces of build.

{{< hint warning >}}
**NOTE:**
The NVCM of the FPGA is a one-time-write only! There is no way of
restoring or reprogramming the FPGA. If you want the possbility to
reprogram the TKey multiple times, consider [programming the SPI
flash](unlocked/spiflash) instead.
{{< /hint >}}

{{< hint warning >}}
**NOTE:**
Only use bitstreams built by yourself or someone you trust. Anyone who
has your bitstream can create copies.
{{< /hint >}}

{{< hint warning >}}
**NOTE:**
If the security bit is **not** set in the FPGA, the TKey should not be
considered secure, and anyone who gets their hands on the TKey can
read out the entire configuration and its secrets.
{{< /hint >}}

We have a Python tool that programs the NVCM. There are two
alternatives to use it: either download our pre-compiled binaries or
run the Python tool directly on your machine.

Connect the TKey Programmer to the computer and place the TKey
correctly in the programming jig. Close the lid by pushing on the
middle of the lid.

{{< tabs "flash nvcm" >}}
{{% tab "Linux" %}}

## 1. Download tools

Download and unzpip or clone the repository next to the
`tillitis-key1` repository

<https://github.com/tillitis/pynvcm>

Navigate to `pynvcm/`

Install the Python interpreter and it's depencencies

```
sudo apt install python3.10-venv
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## 2. Program NVCM

You need permission to speak to the raw USB device of the TKey
Programmer. See [Linux device permissions](tp1/#linux-permissions) to
set it up.

To program NVCM use

```
python3 pynvcm.py --my-design-is-good-enough --write ../tillitis-key1/hw/application_fpga/application_fpga.bin --verify ../tillitis-key1/hw/application_fpga/application_fpga.bin --ignore-blank --secure
```

or another appropriate path to `application_fpga.bin` if you chose
another relative location between the repositories.

When completed, deactivate the venv using

```
deactivate
```

{{% /tab %}}
{{% tab "macOS" %}}

## 1. Download tools

Download and unzpip or clone the repository next to the
`tillitis-key1` repository

<https://github.com/tillitis/pynvcm>

Navigate to `pynvcm/`

Install the Python interpreter and it's depencencies

```
brew install python3.10
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## 2. Program NVCM

To program NVCM use

```
python3 pynvcm.py --my-design-is-good-enough --write ../tillitis-key1/hw/application_fpga/application_fpga.bin --verify ../tillitis-key1/hw/application_fpga/application_fpga.bin --ignore-blank --secure
```

or another appropriate path to `application_fpga.bin` if you chose
another relative location between the repositories.

When completed, deactivate the venv using

```
deactivate
```

{{% /tab %}}
{{% tab "Windows" %}}

## 1. Download tools

Download and unzpip or clone the repository next to the
`tillitis-key1` repository

<https://github.com/tillitis/pynvcm>

Navigate to `pynvcm/`

Install the Python interpreter and it's depencencies
Navigate to `/tillitis-key1/hw/production_tests/` and run

```
winget install Python.Python.3.10
python.exe -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt
```

## 2. Program NVCM

To program NVCM use

```
python.exe pynvcm.py --my-design-is-good-enough --write ..\tillitis-key1\hw\application_fpga\application_fpga.bin --verify ..\tillitis-key1\hw\application_fpga\application_fpga.bin --ignore-blank --secure
```

or another appropriate path to `application_fpga.bin` if you chose
another relative location between the repositories.

when completed, deactivate the venv using

```
deactivate
```

{{% /tab %}}
{{% tab "binaries" %}}

## 1. Download tools

Navigate to `/tillitis-key1/hw/application_fpga/` and download and
unzip the binary for your OS

<https://github.com/tillitis/pynvcm/releases>

## 2. Program NVCM

Use the appropriate executable for your OS, like

```
./pynvcm_0.0.1_linux-amd64 --my-design-is-good-enough --write application_fpga.bin --verify application_fpga.bin --ignore-blank --secure
```

{{% /tab %}}
{{< /tabs >}}

Even if the NVCM is a one-time-write only area, it can be read out if
the security bit is not set. The `--secure` flag to `pynvcm` will set
the
bit to prevent reading NVCM.

Your TKey is now provisioned and ready to be taken out of the
programmer.

## 3. Remove traces of build

It is important to remove all traces of the UDS.

To remove your previously created UDS, you have a few choices. Either
remove these files, with for example `rm`:

```
hw/application_fpga/application_fpga.bin
hw/application_fpga/application_fpga.asc
hw/application_fpga/application_fpga_par.json
hw/application_fpga/synth.v
hw/application_fpga/synth.json
hw/application_fpga/data/uds.hex
```

or natively or your container shell:

```
cd hw/application_fpga
make clean
rm data/uds.hex
```

Once you are done, continue to [casing](unlocked/casing) to assemble
the plastic case.
