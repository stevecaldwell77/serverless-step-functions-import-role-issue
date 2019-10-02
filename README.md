# serverless-step-functions-import-role-issue

Re-creation of bug in [serverless-step-functions](https://github.com/horike37/serverless-step-functions) that prevents the use of `Fn::ImportValue` in assigning an IAM role to a state machine.

Assumes you have the following installed on your system:

  * yarn
  * AWS CLI
  * jq

Step 1: Install packages:

    yarn

Step 2: Deploy a separate stack for managing IAM roles:

    npx cfn-create-or-update \
      --stack-name serverless-step-functions-import-role-issue \
      --template-body file://cfn-iam-roles.yml \
      --capabilities CAPABILITY_IAM \
      --wait

Step 3: Confirm 2 IAM roles were exported (assumes jq is installed):

    aws cloudformation list-exports |
      jq '
        .Exports[] |
        select(.Name == "StateMachineRoleArn" or .Name == "LambdaRoleArn")
      '

Step 4: Attempt to deploy serverless:

    npx serverless deploy

The deployment fails with the following error:

      State machine [hellostepfunc1] is malformed. Please check the README for more info. ValidationError: child "role" fails because ["role" must be a string, "Fn::ImportValue" is not allowed, "Fn::ImportValue" is not allowed]

Step 5: Install older version of serverless-step-functions:

    yarn add --dev serverless-step-functions@2.4

Step 6: Re-attempt to deploy serverless:

    npx serverless deploy

The deployment succeeds.

Step 7: Test state machine:

    aws stepfunctions start-execution \
      --state-machine-arn $(
        aws stepfunctions list-state-machines |
          jq -r '
            .stateMachines[] |
            select(.name == "helloStateMachine") | .stateMachineArn
          '
      )

Step 8: Cleanup

    npx serverless remove

    aws cloudformation delete-stack \
      --stack-name serverless-step-functions-import-role-issue

    aws cloudformation wait stack-delete-complete \
      --stack-name serverless-step-functions-import-role-issue
