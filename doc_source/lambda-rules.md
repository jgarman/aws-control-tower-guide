# AWS Lambda controls<a name="lambda-rules"></a>

**Topics**
+ [\[CT\.LAMBDA\.PR\.2\] Require AWS Lambda function policies to prohibit public access](#ct-lambda-pr-2-description)

## \[CT\.LAMBDA\.PR\.2\] Require AWS Lambda function policies to prohibit public access<a name="ct-lambda-pr-2-description"></a>


|  | 
| --- |
| Comprehensive controls management is available as a preview in all [AWS Regions where AWS Control Tower is offered](https://docs.aws.amazon.com/controltower/latest/userguide/region-how.html)\. These enhanced control capabilities reduce the time required to define and manage the controls you need, to help you meet common control objectives and industry regulations\. No additional charges apply while you use these new capabilities during the preview\. However, when you set up AWS Control Tower, you incur costs for the AWS services that establish your landing zone and implement mandatory controls\. For more information, see [AWS Control Tower pricing](http://aws.amazon.com/controltower/pricing/)\. | 

This control checks whether an AWS Lambda function resource\-based policy prohibits public access\.
+ **Control objective: **Limit network access
+ **Implementation: **AWS CloudFormation Guard Rule
+ **Control behavior: **Proactive
+ **Resource types: **`AWS::Lambda::Permission`
+ **AWS CloudFormation guard rule: ** [CT\.LAMBDA\.PR\.2 rule specification](#ct-lambda-pr-2-rule) 

**Details and examples**
+ For details about the PASS, FAIL, and SKIP behaviors associated with this control, see the: [CT\.LAMBDA\.PR\.2 rule specification](#ct-lambda-pr-2-rule) 
+ For examples of PASS and FAIL CloudFormation Templates related to this control, see: [GitHub](https://docs.aws.amazon.com/https://github.com/aws-samples/aws-control-tower-samples/tree/main/samples/CT.LAMBDA.PR.2) 

**Explanation**

The Lambda function should not be publicly accessible, because it may permit unintended access to your code stored in the function\.

### Remediation for rule failure<a name="ct-lambda-pr-2-remediation"></a>

When setting `Principal` to `*`, provide one of `SourceAccount`, `SourceArn`, or `PrincipalOrgID`\. When setting `Principal` to a service principal \(for example, s3\.amazonaws\.com\), provide one of `SourceAccount` or `SourceArn`\.

The examples that follow show how to implement this remediation\.

#### AWS Lambda Function Policy \- Example One<a name="ct-lambda-pr-2-remediation-1"></a>

AWS Lambda function policy configured with an AWS account ID principal\. The example is shown in JSON and in YAML\.

**JSON example**

```
{
    "LambdaPermission": {
        "Type": "AWS::Lambda::Permission",
        "Properties": {
            "Action": "lambda:InvokeFunction",
            "FunctionName": {
                "Ref": "LambdaFunction"
            },
            "Principal": {
                "Ref": "AWS::AccountId"
            }
        }
    }
}
```

**YAML example**

```
LambdaPermission:
  Type: AWS::Lambda::Permission
  Properties:
    Action: lambda:InvokeFunction
    FunctionName: !Ref 'LambdaFunction'
    Principal: !Ref 'AWS::AccountId'
```

The examples that follow show how to implement this remediation\.

#### AWS Lambda Function Policy \- Example Two<a name="ct-lambda-pr-2-remediation-2"></a>

AWS Lambda function policy configured with a wildcard principal and source account condition\. The example is shown in JSON and in YAML\.

**JSON example**

```
{
    "LambdaPermission": {
        "Type": "AWS::Lambda::Permission",
        "Properties": {
            "Action": "lambda:InvokeFunction",
            "FunctionName": {
                "Ref": "LambdaFunction"
            },
            "Principal": "*",
            "SourceAccount": {
                "Ref": "AWS::AccountId"
            }
        }
    }
}
```

**YAML example**

```
LambdaPermission:
  Type: AWS::Lambda::Permission
  Properties:
    Action: lambda:InvokeFunction
    FunctionName: !Ref 'LambdaFunction'
    Principal: '*'
    SourceAccount: !Ref 'AWS::AccountId'
```

The examples that follow show how to implement this remediation\.

#### AWS Lambda Function Policy \- Example Three<a name="ct-lambda-pr-2-remediation-3"></a>

AWS Lambda function policy configured with a service principal and source ARN condition\. The example is shown in JSON and in YAML\.

**JSON example**

```
{
    "LambdaPermission": {
        "Type": "AWS::Lambda::Permission",
        "Properties": {
            "Action": "lambda:InvokeFunction",
            "FunctionName": {
                "Ref": "LambdaFunction"
            },
            "Principal": "s3.amazonaws.com",
            "SourceArn": {
                "Fn::GetAtt": [
                    "S3Bucket",
                    "Arn"
                ]
            }
        }
    }
}
```

**YAML example**

```
LambdaPermission:
  Type: AWS::Lambda::Permission
  Properties:
    Action: lambda:InvokeFunction
    FunctionName: !Ref 'LambdaFunction'
    Principal: s3.amazonaws.com
    SourceArn: !GetAtt 'S3Bucket.Arn'
```

### CT\.LAMBDA\.PR\.2 rule specification<a name="ct-lambda-pr-2-rule"></a>

```
# ###################################
##       Rule Specification        ##
#####################################
# 
# Rule Identifier:
#   lambda_function_public_access_prohibited_check
# 
# Description:
#   This control checks whether an AWS Lambda function resource-based policy prohibits public access.
# 
# Reports on:
#   AWS::Lambda::Permission
# 
# Evaluates:
#   AWS CloudFormation, AWS CloudFormation hook
# 
# Rule Parameters:
#   None
# 
# Scenarios:
#   Scenario: 1
#     Given: The input document is an AWS CloudFormation or AWS CloudFormation hook document
#       And: The input document does not contain any Lambda permission resources
#      Then: SKIP
#   Scenario: 2
#     Given: The input document is an AWS CloudFormation or AWS CloudFormation hook document
#       And: The input document contains a Lambda permission resource
#       And: 'FunctionUrlAuthType' has been provided with a value of 'NONE'
#      Then: FAIL
#   Scenario: 3
#     Given: The input document is an AWS CloudFormation or AWS CloudFormation hook document
#       And: The input document contains a Lambda permission resource
#       And: 'Principal' has been provided with a wildcard value ('*')
#       And: 'SourceAccount' has not been provided or provided with an empty string value
#       And: 'SourceArn' has not been provided or provided with an empty string value or non-valid local reference
#       And: 'PrincipalOrgID' has not been provided or provided with an empty string value
#      Then: FAIL
#   Scenario: 4
#     Given: The input document is an AWS CloudFormation or AWS CloudFormation hook document
#       And: The input document contains a Lambda permission resource
#       And: 'Principal' has been provided with value that does not match an AWS Account ID, AWS IAM ARN or
#            wildcard value ('*')
#       And: 'SourceAccount' has not been provided or provided with an empty string value
#       And: 'SourceArn' has not been provided or provided with an empty string value or non-valid local reference
#      Then: FAIL
#   Scenario: 5
#     Given: The input document is an AWS CloudFormation or AWS CloudFormation hook document
#       And: The input document contains a Lambda permission resource
#       And: 'Principal' has been provided with an AWS Account ID or AWS IAM ARN value
#      Then: PASS
#   Scenario: 6
#     Given: The input document is an AWS CloudFormation or AWS CloudFormation hook document
#       And: The input document contains a Lambda permission resource
#       And: 'Principal' has been provided with a wildcard value ('*')
#       And: At least one of 'SourceAccount', 'SourceArn' or 'PrincipalOrgID' have been provided with non-empty string
#            values (or a valid local reference for 'SourceArn')
#      Then: PASS
#   Scenario: 7
#     Given: The input document is an AWS CloudFormation or AWS CloudFormation hook document
#       And: The input document contains a Lambda permission resource
#       And: 'Principal' has been provided with value that does not match an AWS Account ID or AWS IAM ARN
#       And: At least one of 'SourceAccount', 'SourceArn' have been provided with non-empty string values (or a valid
#            local reference for 'SourceArn')
#      Then: PASS

#
# Constants
#
let LAMBDA_PERMISSION_TYPE = "AWS::Lambda::Permission"
let AWS_ACCOUNT_ID_PATTERN = /\d{12}/
let AWS_IAM_PRINCIPAL_PATTERN = /^arn:aws[a-z0-9\-]*:iam::\d{12}:.+/

let INPUT_DOCUMENT = this

#
# Assignments
#
let lambda_permissions = Resources.*[ Type == %LAMBDA_PERMISSION_TYPE ]

#
# Primary Rules
#
rule lambda_function_public_access_prohibited_check when is_cfn_template(%INPUT_DOCUMENT)
                                                         %lambda_permissions not empty {
    check(%lambda_permissions.Properties)
        <<
        [CT.LAMBDA.PR.2]: Require AWS Lambda function policies to prohibit public access
        [FIX]: When setting 'Principal' to '*', provide one of 'SourceAccount', 'SourceArn', or 'PrincipalOrgID'. When setting 'Principal' to a service principal (for example, s3.amazonaws.com), provide one of 'SourceAccount' or 'SourceArn'.
        >>
}

rule lambda_function_public_access_prohibited_check when is_cfn_hook(%INPUT_DOCUMENT, %LAMBDA_PERMISSION_TYPE) {
    check(%INPUT_DOCUMENT.%LAMBDA_PERMISSION_TYPE.resourceProperties)
        <<
        [CT.LAMBDA.PR.2]: Require AWS Lambda function policies to prohibit public access
        [FIX]: When setting 'Principal' to '*', provide one of 'SourceAccount', 'SourceArn', or 'PrincipalOrgID'. When setting 'Principal' to a service principal (for example, s3.amazonaws.com), provide one of 'SourceAccount' or 'SourceArn'.
        >>
}

#
# Parameterized Rules
#
rule check(lambda_permission) {
    %lambda_permission {
        # Scenario 2 and 5
        FunctionUrlAuthType not exists or
        FunctionUrlAuthType != "NONE"
    }

    %lambda_permission [
        Principal exists
        Principal == "*"
    ] {
        # Scenario 3 and 6
        SourceAccount exists or
        SourceArn exists or
        PrincipalOrgID exists

        check_is_string_and_not_empty(SourceAccount) or
        check_is_string_or_local_reference(SourceArn) or
        check_is_string_and_not_empty(PrincipalOrgID)
    }

    %lambda_permission [
        Principal exists
        Principal != "*"
        Principal != %AWS_ACCOUNT_ID_PATTERN
        Principal != %AWS_IAM_PRINCIPAL_PATTERN
    ] {
        # Scenario 4 and 7
        SourceAccount exists or
        SourceArn exists

        check_is_string_and_not_empty(SourceAccount) or
        check_is_string_or_local_reference(SourceArn)
    }
}

rule check_is_string_or_local_reference(value) {
    %value {
        check_is_string_and_not_empty(this) or
        check_local_references(%INPUT_DOCUMENT, this)
    }
}

rule check_local_references(doc, reference_properties) {
    %reference_properties {
        'Fn::GetAtt' {
            query_for_resource(%doc, this[0])
                <<Local Stack reference was invalid>>
        } or Ref {
            query_for_resource(%doc, this)
                <<Local Stack reference was invalid>>
        }
    }
}

rule query_for_resource(doc, resource_key) {
    let referenced_resource = %doc.Resources[ keys == %resource_key ]
    %referenced_resource not empty
}

#
# Utility Rules
#
rule is_cfn_template(doc) {
    %doc {
        AWSTemplateFormatVersion exists or
        Resources exists
    }
}

rule is_cfn_hook(doc, RESOURCE_TYPE) {
    %doc.%RESOURCE_TYPE.resourceProperties exists
}

rule check_is_string_and_not_empty(value) {
    %value {
        this is_string
        this != /\A\s*\z/
    }
}
```