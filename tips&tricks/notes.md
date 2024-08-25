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

```
Top 10 reasons I have seen so far for facing CrashLoopBackOff error in Kubernetes:

𝐀𝐩𝐩𝐥𝐢𝐜𝐚𝐭𝐢𝐨𝐧 𝐂𝐨𝐝𝐞 𝐈𝐬𝐬𝐮𝐞𝐬: The container crashes because of a bug or unhandled exception in the application code.

𝐌𝐢𝐬𝐬𝐢𝐧𝐠 𝐃𝐞𝐩𝐞𝐧𝐝𝐞𝐧𝐜𝐢𝐞𝐬: The application fails to start due to missing required dependencies like libraries or config files.

𝐈𝐧𝐜𝐨𝐫𝐫𝐞𝐜𝐭 𝐂𝐨𝐦𝐦𝐚𝐧𝐝 𝐨𝐫 𝐀𝐫𝐠𝐮𝐦𝐞𝐧𝐭𝐬: The container fails to start because of incorrect command or args provided in its specification.

𝐑𝐞𝐬𝐨𝐮𝐫𝐜𝐞 𝐋𝐢𝐦𝐢𝐭𝐬: The container crashes due to insufficient memory or CPU, causing it to be killed by the system (OOMKilled).

𝐂𝐨𝐧𝐟𝐢𝐠𝐮𝐫𝐚𝐭𝐢𝐨𝐧 𝐄𝐫𝐫𝐨𝐫𝐬: The application crashes due to incorrect environment variables, secret keys, or config map configurations.

𝐅𝐢𝐥𝐞 𝐒𝐲𝐬𝐭𝐞𝐦 𝐈𝐬𝐬𝐮𝐞𝐬: The container fails because it can't access necessary files or directories due to incorrect permissions or missing files.

𝐍𝐞𝐭𝐰𝐨𝐫𝐤 𝐈𝐬𝐬𝐮𝐞𝐬: The application fails to start because it cannot reach necessary external services or databases.

𝐏𝐨𝐫𝐭 𝐂𝐨𝐧𝐟𝐥𝐢𝐜𝐭𝐬: The container crashes because the application tries to bind to a port that is already in use or unavailable.

𝐂𝐫𝐚𝐬𝐡 𝐢𝐧 𝐈𝐧𝐢𝐭 𝐂𝐨𝐧𝐭𝐚𝐢𝐧𝐞𝐫𝐬: The main application container never starts due to failure in an init container.

𝐑𝐞𝐚𝐝𝐢𝐧𝐞𝐬𝐬/𝐋𝐢𝐯𝐞𝐧𝐞𝐬𝐬 𝐏𝐫𝐨𝐛𝐞𝐬 𝐌𝐢𝐬𝐜𝐨𝐧𝐟𝐢𝐠𝐮𝐫𝐞𝐝: The container is repeatedly restarted because the liveness probe incorrectly marks it as unhealthy.


```
