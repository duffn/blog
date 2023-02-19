---
title: "InSpec - No AWS Credentials Available"
date: 2021-01-03T00:00:00+00:00
permalink: /inspec-no-aws-credentials-available/
description: "I KNOW my AWS credentials are correct. Why does InSpec tell me they aren't available?"
tags: [aws, inspec, iam]
---

I've used [InSpec](https://docs.chef.io/inspec/) in the past, but it's been some time. I was recently again evaluating it for my current company when I ran into a situation.

I set up an IAM user with the necessary permissions in order to test out InSpec for a proof of concept and added the profile to my `~/.aws/credentials`. I _knew_ the credentials were right because I could test what I wanted with the [AWS CLI](https://aws.amazon.com/cli/) and get a successful response.

```bash
$ aws iam list-mfa-devices --user-name test-user --profile inspec
{
    "MFADevices": [
        {
            "UserName": "test-user",
            "SerialNumber": "arn:aws:iam::12345:mfa/test-user",
            "EnableDate": "2020-01-02T18:41:22Z"
        }
    ]
}
```

So, why then, would InSpec tell me that it couldn't find my AWS credentials for the `inspec` profile when running a simple control?

```ruby
title 'AWS IAM compliance'

control 'iam-1.0' do
  impact 1.0
  title 'Check test-user user has MFA'
  desc 'The test-user user should have MFA enabled'
  describe aws_iam_user(user_name: 'test-user') do
    it { should have_mfa_enabled }
  end
end
```

AWS credentials _are_ available InSpec!

```bash
$ inspec exec aws -t aws://us-east-1/inspec
[2021-01-03T19:25:55-07:00] ERROR: It appears that you have not set your AWS credentials. See https://www.inspec.io/docs/reference/platforms for details.

Profile: AWS InSpec Profile (aws)
Version: 0.1.0
Target:  aws://us-east-1

  ×  iam-1.0: Check test-user user has MFA
     ×  AWS IAM User
     No AWS credentials available


Profile: Amazon Web Services  Resource Pack (inspec-aws)
Version: 1.33.0
Target:  aws://us-east-1

     No tests executed.

Profile Summary: 0 successful controls, 1 control failure, 0 controls skipped
Test Summary: 0 successful, 1 failure, 0 skipped
```

Was I running the correct command to execute my controls? Was I somehow passing my profile to InSpec incorrectly? What was going on?

After some time frustratingly testing numerous different theories, I had another closer look at my `~/.aws/credentials` and noticed that some of my profiles, including my `inspec` profile, looked slightly different than others.

```text
[default]
aws_access_key_id = abcd1234
aws_secret_access_key = abcd1234
region = us-east-1

[otherprofile]
aws_access_key_id = abcd1234
aws_secret_access_key = abcd1234
region = us-east-1

[inspec]
AWS_ACCESS_KEY_ID = abcd1234
AWS_SECRET_ACCESS_KEY = abcd1234
region = us-east-1
```

That can't be it, can it? Is it?

Well it turned out that it was. InSpec was not be able to read my `inspec` profile credentials because `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` were capitalized in my credentials file, because [this is how the Ruby AWS SDK works](https://github.com/aws/aws-sdk-ruby/issues/2458). I copied and pasted from another profile, which I don't recall why I made the keys capitalized, and InSpec couldn't handle it. After changing the keys to `aws_access_key_id` and `aws_secret_access_key`, InSpec found my credentials and I was able to run my controls.

So, if you're having trouble with InSpec telling you `No AWS credentials available` or that `It appears that you have not set your AWS credentials`, check your credentials file and make sure your capitalization is correct.
