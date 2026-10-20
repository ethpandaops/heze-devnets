# focil-devnet-2 spec

:::info
:mega: heze-devnet-2 launch date: 20.10.2026
:::

:::info
:checkered_flag: **Scope.** Next spec devnet on the **Heze** fork. Keeps [EIP-7805 FOCIL](https://eips.ethereum.org/EIPS/eip-7805) as shipped on [focil-devnet-1](https://notes.ethereum.org/@ethpandaops/heze-devnet-1) and adds 2 new EIPs: EIP-8015 and EIP-8365 (CL only). EIP-8198 (Quick Slots) is **not** included; slots stay at 12s.
Client images build from the common `focil-devnet-2` branch.
[frames-devnet-0](https://notes.ethereum.org/@ethpandaops/bogota-devnet-0) (EIP-8141) is kept on a separate devnet track.
:::

## EIP List for focil-devnet-2

| EIP | Title | Layer | Status |
|--------|-----|:-----:|:-----:|
|[EIP-7805](https://eips.ethereum.org/EIPS/eip-7805) | Fork-choice enforced Inclusion Lists (FOCIL) | CL + EL | :heavy_check_mark: |
|[EIP-8015](https://eips.ethereum.org/EIPS/eip-8015) | Remove `deposit` and `eth1data` fields | CL | :new: |
|[EIP-8365](https://eips.ethereum.org/EIPS/eip-8365) | Disallow new 0x00 validators | CL | :new: |

## Testing focus

- **FOCIL regression**: everything from focil-devnet-1 must keep working (IL lifecycle, timing windows, FOCIL × ePBS, IL req/resp).
- **EIP-8015**: blocks without the legacy `deposit`/`eth1data` fields; deposits flowing only through EIP-6110 requests.
- **EIP-8365**: deposits that would create new validators with 0x00 withdrawal credentials are rejected; existing 0x00 validators keep working.

### Specs & Tests

**Consensus Layer**

- [x] [v1.7.0-beta.4](https://github.com/ethereum/consensus-specs/releases/tag/v1.7.0-beta.4) :heavy_check_mark:

**Execution Layer**

- [x] [focil-devnet@v0.3.0](https://github.com/ethereum/execution-specs/releases/tag/tests-focil-devnet%40v0.3.0) :heavy_check_mark:

### Open PRs

FOCIL PRs are tracked on [focil-devnet-1](https://notes.ethereum.org/@ethpandaops/heze-devnet-1#Open-PRs).

**Consensus Specs**

- [PR-5655 - Add EIP-8015 to Heze](https://github.com/ethereum/consensus-specs/pull/5655) Merged :heavy_check_mark:
- [PR-5713 - Add EIP-8365 to Heze](https://github.com/ethereum/consensus-specs/pull/5713) Merged :heavy_check_mark:
- [PR-5721 - Sign inclusion lists with the inclusion list epoch's fork version](https://github.com/ethereum/consensus-specs/pull/5721) Merged :heavy_check_mark:
- [PR-5724 - Avoid mutating the inclusion list store in read-only lookups](https://github.com/ethereum/consensus-specs/pull/5724) Merged :heavy_check_mark:
- [PR-5727 - Set the IL deadline to 9s into the slot](https://github.com/ethereum/consensus-specs/pull/5727) Merged :heavy_check_mark:

## Local testing

Kurtosis example:

```yaml
participants_matrix:
  el:
    - el_type: ethrex
      el_image: ethpandaops/ethrex:focil-devnet-2
    - el_type: besu
      el_image: ethpandaops/besu:focil-devnet-2
    - el_type: nethermind
      el_image: ethpandaops/nethermind:focil-devnet-2
    - el_type: reth
      el_image: ethpandaops/reth:focil-devnet-2
  cl:
    - cl_type: lighthouse
      cl_image: ethpandaops/lighthouse:focil-devnet-2
    - cl_type: lodestar
      cl_image: ethpandaops/lodestar:focil-devnet-2
    - cl_type: teku
      cl_image: ethpandaops/teku:focil-devnet-2

network_params:
  preset: minimal
  gloas_fork_epoch: 0
  heze_fork_epoch: 1

additional_services:
  - spamoor
  - dora

global_log_level: debug
```

---

Are you looking for frames? https://notes.ethereum.org/@ethpandaops/bogota-devnet-0
