# Overview

Snapshot X is designed to be highly modular to provide maximum flexibility for DAOs. Each DAO will deploy their own space contract via the factory and will add the desired modules as settings in the space. 

Snapshot X modules are one of 3 categories: Authenticators, Voting Strategies, and Execution Strategies. Each space needs at least one of each type in order to create a proposal. In general the modules act as library contracts where the same instance is shared by all spaces that use it. Therefore to use a particular module in your space, you simply to find the address of the library and then whitelist it in your space. Standard modules will be heavily audited so that users of them can have high trust in their functionality being secure. However Snapshot X is competely permissionless and DAOs are free to write their own modules and use them as they see fit. 

Here is a diagram showcasing how the contracts that make up Snapshot X fit together along with various actions users can make to create proposals, vote, execute proposals, and update settings.


{% embed url="https://whimsical.com/snapshot-x-core-5ryHXBELMwg9L9nA6rMeDs" %}
