For jenkins

```
sh "aws sts assume-role --role-arn arn:aws:iam::account-id:role/role-name --role-session-name session-name --duration-seconds 3600 --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' --output text | tr '\t' '\n' > env_vars"

sh "cp env_vars temp_vars"

sh "echo 'export AWS_ACCESS_KEY_ID='$(awk 'NR==1' temp_vars) > env_vars"
sh "echo 'export AWS_SECRET_ACCESS_KEY='$(awk 'NR==2' temp_vars) >> env_vars"
sh "echo 'export AWS_SESSION_TOKEN='$(awk 'NR==3' temp_vars) >> env_vars"

sh "rm temp_vars"


sh "source env_vars"


sh "aws eks --region region update-kubeconfig --name cluster-name --role-arn arn:aws:iam::account-id:role/role-name"


sh "echo $KUBECONFIG"


sh "kubectl apply -f path/to/application.yaml"

```