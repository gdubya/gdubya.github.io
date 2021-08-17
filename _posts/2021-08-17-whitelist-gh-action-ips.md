---
layout: post
title:  "Whitelist a Github Action"
date:   2021-08-17 19:41:00 +0200
categories: azure cosmos github
---

I'm using Github Actions to automatically archive journey data from my car. For now I'm writing the data to a local sqlite database stored in a private Git repo, but I'd like to push it to a "remote" database so that the data is more easily accessible. The plan is to use Github Actions to run a script to write the data to a [Azure CosmosDB "Free Tier"][cosmos-free-tier] instance.

I've configured the Cosmos firewall to only allow access from whitelisted IPs, so I will need to add all the Github Action IP ranges to this whitelist. These IPs can be found at [api.github.com/meta][api-github-meta], under "actions". I'll run the script locally this time, but if I need to automate this in an Action as well then I could use [the Azure Login Action][azure-login-action] to authenticate before running the CLI in a workflow.

Here's the script for fetching the list of CIDR ranges (taking only the valid IPv4 ranges, since it doesn't look like IPv6 is supported yet) and then passing the list as a parameter to the "az cosmosdb update" command:

{% highlight bash %}
    # Build the iplist
    for ip in $(curl https://api.github.com/meta | jq -r '.actions | .[]'); do
    if [[ $ip =~ ^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/[0-9][0-9]$ ]]; then
        iplist="$ip,$iplist"
    fi
    done;

    # Remove the trailing comma
    iplist=${iplist: : -1}

    # Update Cosmos
    az cosmosdb update --resource-group $rgname --name $dbname --ip-range-filter $iplist
{% endhighlight %}

[cosmos-free-tier]: https://docs.microsoft.com/en-us/azure/cosmos-db/free-tier
[api-github-meta]: https://api.github.com/meta
[azure-login-action]: https://github.com/Azure/login
