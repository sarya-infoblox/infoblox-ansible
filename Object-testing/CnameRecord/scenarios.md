# CNAME Record (nios_cname_record) Test Scenarios

## Prerequisites

| Dependency | Default? | Handling |
|---|---|---|
| DNS view `default` | Yes — every NIOS Grid ships with it | Used as-is; no creation needed |
| DNS zone `ansible.test` (in `default` view) | **No** | Every playbook that needs it creates it with `nios_zone state: present` before the test task. Idempotent across runs. |
| DNS view `ansible-test-view` | **No** | Created by scenario 11 only, cleaned up at end of that playbook. |
| DNS zone `ansible.test` in `ansible-test-view` | **No** | Created by scenario 11 only. |

All playbooks follow this structure:
1. Ensure zone (and view if needed) exists — idempotent
2. Cleanup any leftover objects from prior runs
3. Execute the scenario task(s)
4. Assert `changed` / `not changed` with `success_msg` / `fail_msg`
5. Cleanup created objects

---

## Scenario Table

| # | Playbook | Description | Key Assertions |
|---|---|---|---|
| 01 | `01_create_basic.yml` | Create CNAME with valid alias + canonical; verify idempotent on second run | `changed=true` first run; `changed=false` second run |
| 02 | `02_create_trailing_dot.yml` | Create CNAME with trailing-dot canonical; re-apply without dot → must be idempotent (NIOS normalizes) | Second task `changed=false` |
| 03 | `03_update_canonical.yml` | Create CNAME, then update canonical to a different target | Create `changed=true`; update `changed=true`; second update `changed=false` |
| 04 | `04_rename_alias.yml` | Rename alias using `{old_name, new_name}` dict | Rename task `changed=true`; alias under new name present, old name absent |
| 05 | `05_update_comment_extattrs.yml` | Create CNAME, update comment and extattrs | Create `changed=true`; update `changed=true`; re-apply same `changed=false` |
| 06 | `06_ttl_set_unset.yml` | Set explicit TTL, verify stored; omit TTL on re-apply, observe use_ttl gap behavior | TTL set task `changed=true`; re-apply same TTL `changed=false` |
| 07 | `07_delete_existing.yml` | Delete a CNAME that exists | Delete task `changed=true` |
| 08 | `08_delete_absent_idempotent.yml` | Delete a CNAME that does not exist (state: absent) | Task `changed=false`, no error |
| 09 | `09_idempotency_create.yml` | Full idempotency: create → re-apply with all same params | Second run: every task `changed=false` |
| 10 | `10_check_mode.yml` | Create in `--check` mode; verify no real object created; then real create succeeds | Check-mode task `changed=true` (prediction); real create `changed=true` |
| 11 | `11_split_horizon.yml` | Same alias FQDN in two different views with different targets | Both creates `changed=true`; each view stores its own target |
| 12 | `12_cname_a_conflict.yml` | Create A record at a name, then attempt CNAME at same name → WAPI must reject | CNAME create task `failed=true` |
| 13 | `13_nonexistent_zone_error.yml` | Create CNAME in a zone that does not exist → WAPI must reject | Task `failed=true` |
| 14 | `14_missing_required_params.yml` | Invoke module without required `canonical` param → AnsibleModule must fail | Task `failed=true` with param error |
| 15 | `15_ea_filter_update.yml` | Create CNAME with EAs; update EA value; verify EA update idempotent | EA update `changed=true`; re-apply same EA `changed=false` |
| 16 | `16_max_fqdn_length.yml` | Alias at 253-char FQDN (label-boundary safe); create and delete | Create `changed=true`; delete `changed=true` |
| 17 | `17_case_normalization.yml` | Create alias in UPPERCASE; re-apply with lowercase → must be idempotent (NIOS normalizes to lowercase) | Second run `changed=false` |
| 18 | `18_use_ttl_gap.yml` | Set TTL via module, verify WAPI stores it; document `use_ttl` field is not exposed | Exploratory: records observed TTL behavior; flags gap |
