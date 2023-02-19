---
title: "Get All RDS Instances (or Any Resource) By Region and Profile"
date: 2020-12-20T00:00:00+00:00
permalink: /get-rds-instances-by-region-profile/
description: "Get AWS Resources Across all Regions and Profiles."
tags: [aws, aws-cli, rds, aws-vault]
---

I needed to get a list of all RDS instances, but not just in a single region or profile, but across all AWS regions and across all of my AWS profiles.

You can do this with RDS (and any resource really), using the CLI and a couple of loops in bash.

See the comments below for additional information and happy scripting!

```bash
echo "Getting all RDS instances in all regions for all accounts."
echo "This could take awhile..."

# This assumes you have multiple profiles setup in your AWS credentials for the CLI.
# https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html
PROFILES=(myprofile1 myprofile2 myprofile3)

# echo the fields that we're extracting from the returned JSON so we have a header in our CSV file.
echo '"Profile", "Region", "DBInstanceIdentifier", "DBName", "DBInstanceClass", "Engine", "AvailabilityZone", "DBInstanceStatus", "Endpoint".Address, "AllocatedStorage"'
# Get all of the regions that are enabled on our account.
for region in $(aws ec2 describe-regions --output text | cut -f4); do
  for profile in "${PROFILES[@]}"; do
    # Extract the fields that we want from the response and include our profile and region as arguments so we can get them in the output as well.
    # Note that I'm using `aws-vault` here and you should too! https://github.com/99designs/aws-vault
    aws-vault exec "$profile" -- aws rds describe-db-instances --region "$region" | \
      jq -r --arg PROFILE "$profile" --arg REGION "$region" '.DBInstances[] | [$PROFILE, $REGION, .DBInstanceIdentifier, .DBName, .DBInstanceClass, .Engine, .AvailabilityZone, .DBInstanceStatus, .Endpoint.Address, .AllocatedStorage] | @csv'
  done
done
```
