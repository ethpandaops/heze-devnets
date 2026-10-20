# heze-devnet-3 spec

:::info
:mega: heze-devnet-3 launch date: 20.10.2026
:::

:::info
:checkered_flag: **Scope.** Next spec devnet on the **Heze** fork. Keeps everything from [heze-devnet-2](https://notes.ethereum.org/@ethpandaops/heze-devnet-2) ([EIP-7805 FOCIL](https://eips.ethereum.org/EIPS/eip-7805), EIP-8015, EIP-8365) and adds EIP-8198 (CL + EL): slots go from 12s to **10s** at the Heze fork.
Client images build from the common `heze-devnet-3` branch.
[frames-devnet-0](https://notes.ethereum.org/@ethpandaops/bogota-devnet-0) (EIP-8141) is kept on a separate devnet track.
:::

## EIP List for heze-devnet-3

| EIP | Title | Layer | Status |
|--------|-----|:-----:|:-----:|
|[EIP-7805](https://eips.ethereum.org/EIPS/eip-7805) | Fork-choice enforced Inclusion Lists (FOCIL) | CL + EL | :heavy_check_mark: |
|[EIP-8015](https://eips.ethereum.org/EIPS/eip-8015) | Remove `deposit` and `eth1data` fields | CL | :heavy_check_mark: |
|[EIP-8365](https://eips.ethereum.org/EIPS/eip-8365) | Disallow new 0x00 validators | CL | :heavy_check_mark: |
|[EIP-8198](https://eips.ethereum.org/EIPS/eip-8198) | Quick Slots | CL + EL | :new: |

## Testing focus

- **heze-devnet-2 regression**: FOCIL (IL lifecycle, timing windows, FOCIL × ePBS, IL req/resp), EIP-8015 and EIP-8365 must keep working.
- **EIP-8198**: 12s → 10s slots derived from the fork configuration; IL deadlines and ePBS payload timing at the new slot length. On the EL, base fee max change of 5/48 and the new blob schedule (target 12, max 18, update fraction 12018519; provisional, needs a matching CL `BLOB_SCHEDULE` entry).

### Specs & Tests

**Consensus Layer**

- [ ] [v1.7.0-beta.5](https://github.com/ethereum/consensus-specs/releases/tag/v1.7.0-beta.5) :hourglass:

**Execution Layer**

- [ ] Next focil-devnet test release including EIP-8198 ([PR-3703](https://github.com/ethereum/execution-specs/pull/3703)); FOCIL-only baseline is [focil-devnet@v0.3.0](https://github.com/ethereum/execution-specs/releases/tag/tests-focil-devnet%40v0.3.0) :hourglass:

### Open PRs

FOCIL PRs are tracked on [focil-devnet-1](https://notes.ethereum.org/@ethpandaops/heze-devnet-1#Open-PRs).

**Consensus Specs**

- [PR-5592 - Add quick slots (EIP-8198)](https://github.com/ethereum/consensus-specs/pull/5592) Merged :heavy_check_mark:
- [PR-5710 - Derive EIP-8198 slot timing from fork configuration](https://github.com/ethereum/consensus-specs/pull/5710) Merged :heavy_check_mark:
- [PR-5717 - Return slot durations as tuples in `get_slot_durations`](https://github.com/ethereum/consensus-specs/pull/5717) Merged :heavy_check_mark:
- [PR-5718 - Add EIP-8198 to Heze](https://github.com/ethereum/consensus-specs/pull/5718) Open :exclamation:

**Execution Spec PRs**

- [PR-3703 - feat(spec-specs,test-forks,tests): implement EIP-8198 quick slots](https://github.com/ethereum/execution-specs/pull/3703) Merged :heavy_check_mark:

## Local testing

Kurtosis example:

```yaml
participants_matrix:
  el:
    - el_type: ethrex
      el_image: ethpandaops/ethrex:heze-devnet-3
    - el_type: besu
      el_image: ethpandaops/besu:heze-devnet-3
    - el_type: nethermind
      el_image: ethpandaops/nethermind:heze-devnet-3
    - el_type: reth
      el_image: ethpandaops/reth:heze-devnet-3
  cl:
    - cl_type: lighthouse
      cl_image: ethpandaops/lighthouse:heze-devnet-3
      supernode: true
    - cl_type: lodestar
      cl_image: ethpandaops/lodestar:heze-devnet-3
    - cl_type: teku
      cl_image: ethpandaops/teku:heze-devnet-3

network_params:
  gloas_fork_epoch: 0
  heze_fork_epoch: 1
  slot_duration_ms_heze: 10000

additional_services:
  - spamoor
  - dora

global_log_level: debug

dora_params:
  image: ethpandaops/dora:master
```

---

Are you looking for frames? https://notes.ethereum.org/@ethpandaops/bogota-devnet-0
