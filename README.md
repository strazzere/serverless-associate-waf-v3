# serverless-associate-waf-v3

Associate a regional WAF with the AWS API Gateway used by your Serverless stack.

This is based off [serverless-associate-waf](https://github.com/mikesouza/serverless-associate-waf), essentially all the same code and retains the same licensing. The fork is due to it (seemingly) being abandoned and having many old dependencies. For a project I was working on I needed to update a few things and wanted to upgrade it to the Serverless Plugin architecture v3, thus the naming.

Main changes:
 - Upgrade to Serverless Plugin v3
 - Utilize v3 logging methodology
 - Utilize `getAccountInfo` to get the `partition` of the `arn` (required for govcloud to work)
 - Upgrade dependencies and tests
 - When failing to associate or disassociate the waf, throw a `ServerlessError` to stop the deploy vs silently failing and allowing to proceed

## Install

`npm install serverless-associate-waf-v3 --save-dev`

## Configuration

Add the plugin to your `serverless.yml`:

```yaml
plugins:
  - serverless-associate-waf-v3
```

### Associating a Regional WAF with the API Gateway

Add your custom configuration:

```yaml
custom:
  associateWaf:
    name: myRegionalWaf
    version: Regional #(optional) Regional | V2
```

| Property | Required | Type     | Default | Description                                                    |
|----------|----------|----------|---------|----------------------------------------------------------------|
| `name`   |  `true`  | `string` |         | The name of the regional WAF to associate the API Gateway with |
| `version`|  `false` | `string` | `Regional`| The AWS WAF version to be used|

You will also need to add extra permissions to the user if it does not already include the following - consider this an example only, you can restrict it further:
```yaml
provider:
  name: aws
  runtime: nodejs18.x
  region: us-west-1
  endpointType: REGIONAL
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - apigateway:SetWebACL
          Resource:
            - 'arn:aws:apigateway:us-west-1::/*/*'
        - Effect: Allow
          Action:
            - wafv2:ListWebACLs
            - wafv2:AssociateWebACL
            - wafv2:DisassociateWebACL
            - wafv2:GetWebACLForResource
          Resource:
            - 'arn:aws:wafv2:us-west-1:ACCOUNTNUMBER:regional/webacl/*/*'
```

### Disassociating a Regional WAF from the API Gateway

Remove the `name` property from your custom configuration but keep the `version` if specified, and then deploy the application. The plugin must stay in the plugins list of `serverless.yml` in order for the WAF to be disassociated.

## Usage

Configuration of your `serverless.yml` is all you need.

There are no custom commands, just run: `sls deploy`
