# Inventory

## Local Inventory

- OPI adopted [SMBIOS](https://www.dmtf.org/standards/smbios) for DPUs and IPUs
- SMBIOS is used for local access inside the DPUs and IPUs to get BIOS and System information about DPUs and IPUs
- SMBIOS is a standard way to get similar information from servers, so DPUs and IPUs adoption make sense
- On Linux one can access SMBIOS info locally via `dmidecode` or `/sys/class/dmi/id/`, as an example
- SMBIOS provides those types of information:
  - bios
  - system
  - baseboard
  - chassis
  - processor
  - memory
  - cache
  - connector
  - slot
- OPI will define mandatory tables and fields for OPI compliance.
- OPI will produce smbios-validation-tool like [google](https://github.com/google/smbios-validation-tool)
- Check more info in Provisioning TWG <https://github.com/opiproject/opi-prov-life>

Example from DELL:

```bash
$ dmidecode -t system

Handle 0x0100, DMI type 1, 27 bytes
System Information
        Manufacturer: Dell Inc.
        Product Name: PowerEdge R750
        Version: Not Specified
        Serial Number: 3Z7CMH3
        UUID: 4c4c4544-005a-3710-8043-b3c04f4d4833
        Wake-up Type: Power Switch
        SKU Number: SKU=NotProvided;ModelName=PowerEdge R750
        Family: PowerEdge
```

Example from Nvidia:

```bash
$ dmidecode -t system

Handle 0x0001, DMI type 1, 27 bytes
System Information
        Manufacturer: https://www.mellanox.com
        Product Name: BlueField SoC
        Version: 1.0.0
        Serial Number: Unspecified System Serial Number
        UUID: 2e3bc1d1-e205-4830-a817-968ed1978bac
        Wake-up Type: Power Switch
        SKU Number: Unspecified System SKU
        Family: BlueField
```

Example from Marvell:

```bash
$ dmidecode -t system

Handle 0x0001, DMI type 1, 27 bytes
System Information
        Manufacturer: Marvell
        Product Name: ebb106
        Version: unknown
        Serial Number: unknown
        UUID: ea04f2b5-0b7a-4ba9-9b46-13ea9f5f1b95
        Wake-up Type: Power Switch
        SKU Number: Not specified
        Family: Octeon 10
```

Example from Intel:

```bash
$ dmidecode -t system

Handle 0x0001, DMI type 1, 27 bytes
System Information
        Manufacturer: Intel
        Product Name: Intel(R) Infrastructure Processing Unit (Intel(R) IPU)
        Version: A0
        Serial Number: -------------
        UUID: 8a95d198-7f46-11e5-bf8b-08002704d48e
        Wake-up Type: Power Switch
        SKU Number: -------------
        Family: -------------
```

## Remote Network Inventory

- Redfish
  - what if there is no NIC BMC and no IPU IMC ? Run redfish server on the ARM cores
- gRPC
  - add new gRPC service and RPC calls for inventory query
  - see proto definition [here](https://github.com/opiproject/opi-api/blob/main/common/v1/inventory.proto)
  - see example implementation [here](https://github.com/opiproject/opi-smbios-bridge)

## Remote Host/BMC Inventory

Host host/platform BMC access inventory information from the DPU/IPU ?

- NC-SI
- PLDM
- VPD
- other...

VPD Example from Nvidia:

```bash
$ lspci -s ca:00.0 -vvv | grep -A 11 "Vital Product Data"
        Capabilities: [48] Vital Product Data
                Product Name: BlueField-2 E-Series SmartNIC 100GbE/EDR VPI Dual-Port QSFP56, PCIe Gen4 x16, Crypto Enabled, 16GB on-board DDR, FHHL
                Read-only fields:
                        [PN] Part number: MBF2M516A-EEEOT
                        [EC] Engineering changes: A1
                        [V2] Vendor specific: MBF2M516A-EEEOT
                        [SN] Serial number: MT2022X19080
                        [V3] Vendor specific: 805647c144a9ea1180000c42a198b662
                        [VA] Vendor specific: MLX:MN=MLNX:CSKU=V2:UUID=V3:PCI=V0:MODL=BF2M526A
                        [V0] Vendor specific: PCIeGen4 x16
                        [RV] Reserved: checksum good, 1 byte(s) reserved
                End
```

## Leftovers

this page talks about common inventory information information provided by DPU vendors in a same format

- Questions:
  - Where does it run?
    - service/container running on ARM cores?
    - DPU firmware?
    - DPU Bootloaders?
    - DPU BMC?
    - External, as a control plane component?
  - Is it always available/running?
    - if this is e.g. a UEFI application then it would only be available pre-boot.
  - What information does it provide ?
    - Vendor, SN/PN, ...
    - Credentials
    - Virtual location in the host (mother board slot or PCIe BDF?)
    - Capabilities (cou, mem, offloads,...)
    - What protocols and/or privisoning methods are supported
    - What DPU belongs to what Host ?
  - Does it need external sources of information to perform its function?
    - mapping to a host / host PCI topology is probably not possible from the perspective of a PCI device.
  - What protocol and what format is used?
    - GraphQL? Rest ? gRPC?
    - JSON/XML/Protobuf?
    - in FW we can't use TCP probably, so look into ICMP
  - Can we simplify the approach?
    - If we break this into a data collector application and a data store/query service as two separate entities:
      - Their actual run times can be non contemporaneous (e.g. first one runs periodically, or when things change, or remotely, ...).
      - Also a benefit exists here where the latter can be a generic OPI software, while the former is vendor-specific by nature.

this is not the end... just pausing here the doc...
