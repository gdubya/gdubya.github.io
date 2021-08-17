---
layout: post
title:  "Whitelist a Github Action Runner IP in Azure CosmosDB"
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

    # Update Cosmos - may take a few minutes to execute
    az cosmosdb update --resource-group $rgname --name $dbname --ip-range-filter $iplist
{% endhighlight %}

Once the az command is finished then you should see a long list of CIDR ranges under the firewall settings.

![CosmosDB Firewall Settings]({{ site.url }}/images/cosmos-firewall-whitelist.png)

It's a long list of ranges, but it's better than exposing the database to the _entire_ Internet.

[cosmos-free-tier]: https://docs.microsoft.com/en-us/azure/cosmos-db/free-tier
[api-github-meta]: https://api.github.com/meta
[azure-login-action]: https://github.com/Azure/login
