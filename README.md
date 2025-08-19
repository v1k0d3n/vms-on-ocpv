# VMs on OpenShift Virtualization Helm Chart

This Helm Chart provides a simple way to deploy multiple Virtual Machines on OpenShift Virtualization using KubeVirt. It's designed to be flexible and configurable, allowing you to deploy up to 100 VMs with consistent naming and MAC address patterns.

## Features

- **Scalable VM Deployment**: Deploy 1-100 VMs with consistent naming patterns
- **Flexible Disk Configuration**: Configure multiple disks with custom sizes and boot orders
- **MAC Address Management**: Automatic MAC address generation using hex-based numbering
- **Network Configuration**: Support for Multus network attachments

- **Consistent Labeling**: Apply common labels and annotations to all resources

## Prerequisites

- OpenShift Virtualization (KubeVirt) installed and operational
- NetworkAttachmentDefinition configured (e.g., `default/vlan3`)
- StorageClass available for VM disks
- Helm 3.x installed

## Installation

1. To install the Chart, you can use the following example:

   ```bash
   helm repo add v1k0d3n https://v1k0d3n.github.io/charts
   helm repo update

   helm search repo v1k0d3n --devel

   helm install vms-on-ocpv v1k0d3n/vms-on-ocpv \
     --namespace jinkit-kvm \
     --create-namespace
   ```

## Configuration

The following table lists the configurable parameters and their default values:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `common.metadata.labels` | Labels applied to all resources | `environment: development`, `owner: v1k0d3n`, etc. |
| `common.metadata.annotations` | Annotations applied to all resources | `description: "This cluster was created using..."` |
| `VirtualMachine.baseName` | Base name for VM naming | `ztp-jinkit-kvm` |
| `VirtualMachine.count` | Number of VMs to create (0-99) | `3` |
| `VirtualMachine.resources.cpuCount` | Number of CPU cores per VM | `20` |
| `VirtualMachine.resources.memoryCount` | Memory allocation per VM | `48Gi` |
| `VirtualMachine.resources.disks` | Disk configuration list | See values.yaml |
| `VirtualMachine.resources.storageClass` | StorageClass for VM disks | `lvms-vg1-immediate` |
| `VirtualMachine.resources.network.networkName` | NetworkAttachmentDefinition | `default/vlan3` |
| `VirtualMachine.resources.network.baseMAC` | Base MAC address prefix | `00:50:56:03:17` |
| `VirtualMachine.template.architecture` | VM architecture | `amd64` |
| `VirtualMachine.template.machineType` | Machine type | `pc-q35-rhel9.6.0` |
| `VirtualMachine.template.runStrategy` | VM run strategy | `Always` |

### Example Configuration

```yaml
common:
  metadata:
    labels:
      environment: production
      owner: team-a
      project: ztp-deployment
    annotations:
      description: "This cluster was created using the vms-on-ocpv Helm chart"

VirtualMachine:
  baseName: ztp-prod-cluster
  count: 5
  resources:
    cpuCount: 16
    memoryCount: 32Gi
    disks:
      - name: disk1
        diskSize: 100Gi
        bootOrder: 2
      - name: disk2
        diskSize: 200Gi
    storageClass: fast-ssd
    network:
      networkName: default/vlan10
      baseMAC: "00:50:56:04:18"
```

## MAC Address Generation

The chart automatically generates MAC addresses for each VM using the following pattern:
- Base MAC: `00:50:56:03:17` (configurable)
- VM number: `00`, `01`, `02`, etc. (based on count)
- Final MAC: `00:50:56:03:17:00`, `00:50:56:03:17:01`, etc.

For VMs numbered 10-99, the hex representation is used:
- VM 10 → MAC: `00:50:56:03:17:0A`
- VM 99 → MAC: `00:50:56:03:17:63`

## Usage

### Deploy with Custom Values

```bash
helm install my-vms . -n jinkit-kvm -f custom-values.yaml
```

### Upgrade Existing Deployment

```bash
helm upgrade my-vms . -n jinkit-kvm -f updated-values.yaml
```

### Uninstall

```bash
helm uninstall my-vms -n jinkit-kvm
```

## VM Naming Convention

VMs are named using the pattern: `<baseName>-<number>`
- Example: `ztp-jinkit-kvm-00`, `ztp-jinkit-kvm-01`, `ztp-jinkit-kvm-02`

## Network Configuration

The chart supports Multus network attachments. Ensure your NetworkAttachmentDefinition exists before deploying:

```yaml
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: vlan3
  namespace: default
spec:
  config: |
    {
      "cniVersion": "0.3.1",
      "name": "ens6f1-localnet",
      "type": "ovn-k8s-cni-overlay",
      "topology": "localnet",
      "vlanID": 3,
      "netAttachDefName": "default/vlan3"
    }
```

## Troubleshooting

### Check VM Status

```bash
oc get vms -n <namespace>
oc describe vm <vm-name> -n <namespace>
```

### Check DataVolumes

```bash
oc get datavolumes -n <namespace>
oc describe datavolume <dv-name> -n <namespace>
```

### Check Network Attachments

```bash
oc get network-attachment-definitions -n default
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with `helm template .`
5. Submit a pull request

## License

This project is licensed under the Apache Version 2.0 license - see the [LICENSE](../LICENSE) file for details.
