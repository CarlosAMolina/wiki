# Step Function

## AWS cli

[Referencia](https://docs.aws.amazon.com/cli/latest/reference/stepfunctions/).

Listar sfs en una cuenta:

```bash
# https://stackoverflow.com/questions/74908738/aws-cli-query-and-filter-for-step-functions
aws stepfunctions list-state-machines --query 'stateMachines[?contains(name,`dev`)]|[*].stateMachineArn' --region eu-west-1 --output yaml
```

Mostrar definición de un step function ([documentación](https://docs.aws.amazon.com/cli/latest/reference/stepfunctions/describe-state-machine.html)):

```bash
aws stepfunctions describe-state-machine --region eu-west-1 --output text --query definition --state-machine-arn arn:aws:states:eu-west-1:012345678901:stateMachine:foo
```

Script para varios step functions:

```bash
#!/bin/bash

set -e # Exit if any command fails.

aws_account_id="012345678901"
aws_region="eu-west-1"
sf_names=(
"foo"
"bar"
)

mkdir $aws_account_id
for sf_name in ${sf_names[@]}
do
  sf_arn=arn:aws:states:$aws_region:$aws_account_id:stateMachine:$sf_name
  echo "Start exporting ARN: $sf_arn"
  aws stepfunctions describe-state-machine --region $aws_region --output text --query definition --state-machine-arn $sf_arn > $aws_account_id/$sf_name.json
done
```

## Referencias

- [Input and Output Processing in Step Functions - AWS Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-input-output-filtering.html).
- [Step Functions event simulator](https://us-east-1.console.aws.amazon.com/states/home?region=us-east-1#/simulator).

