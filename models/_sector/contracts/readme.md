# Runbook to add a new chain

This is a long process. Ideally we figure out a solution where we don't need to do one per chain, and instead can unify all chains (please help).

1. Add a new file for the chain in `contracts/chains/contract_creator_project_mapping/`
2. Create dummy legacy model
3. Add this new chain alias to `contracts/chains/contract_creator_project_mapping/contracts_contract_creator_project_mapping_schema.sql`
4. Add the ref to the model to `contracts/contracts_contract_mapping.sql`

5. Add a new file for the chain in `contracts/chains/find_self_destruct_contracts/`
6. Create dummy legacy model
7. Add this new chain alias to `contracts/chains/find_self_destruct_contracts/contracts_find_self_destruct_contracts_schema.sql`
8. Add the ref to the model to `contracts/contracts_optimism_self_destruct_contracts.sql`

{% docs contract_mapping %}

# Contract Mapping
This repository contains all the source code for `contracts.contract_mapping` that you can use to join contract addresses on each chain to pull our mapped project names in the `contract_project` field.

Mappings are derived from either mapping the contract deployer address to a project name `contracts.contract_creator_address_list`, reading from Dune's decoded contracts or manually overriding when the creator is not deterministic (i.e. the deployer creates contracts for multiple projects). We also use `contracts.project_name_mappings` to consolidate variations in project names. You can find the dependency graph generated by dbt [here](https://spellbook-docs.dune.com/#!/model/model.spellbook.contracts_contract_mapping).

## How to Use
Below is a simplified example of using contract mapping table to get apps creating most transactions in the last 24 hours.

```sql
select 
  cm.contract_project
  ,count(*) as num_txs
  ,sum(t.l1_fee + t.gas_used * t.gas_price)/1e18 as eth_fees
from optimism.transactions as t 
join contracts.contract_mapping as cm
  on t.to = cm.contract_address
  AND cm.blockchain = 'optimism'
  and not is_self_destruct
where 
  t.block_time > now() - interval '24 hours'
  and t.success
group by 1
order by 2 desc
```

## How To Contribute
1. Follow the format and add the creator address and contract project to `contracts_contract_creator_address_list.sql`
2. If you want to enforce consistent project name mapping other than Dune's decoded contract one, then add the name mapping to `contracts_project_name_mappings.sql`
3. Submit the pull request (PR) and feel free to tag [@MSilb7](https://github.com/MSilb7) or [@chuxinh](https://github.com/chuxinh) if there's any questions
4. Once the PR is merged, check on Dune if your work is correctly reflected 🔴✨

{% enddocs %}