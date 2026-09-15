# 08 — Amazon RDS for PostgreSQL step-by-step tutorial

## Goal

Create a small Amazon RDS for PostgreSQL DB instance, connect to it securely from AWS CloudShell, create and query a table, observe an intentional SQL error, reconnect to prove that the data persists, and delete all resources created by the tutorial.

**Difficulty:** Beginner  
**Time:** About 30–40 minutes, including database creation time  
**AWS charges:** An RDS DB instance can incur compute, database storage, backup storage, and data-transfer charges while it exists. Free Tier eligibility depends on the account, Region, instance class, storage, and total usage. Use the smallest Free Tier-eligible class shown in your Console and complete the cleanup section immediately after the lab.

## What you will create

| Item | Tutorial value |
| --- | --- |
| Database engine | PostgreSQL |
| DB instance identifier | `tutorial-postgres-db` |
| Initial database name | `tutorialdb` |
| Master username | `tutorialadmin` |
| DB instance class | `db.t4g.micro` or `db.t3.micro` if marked Free Tier eligible |
| Deployment | Single DB instance in one Availability Zone |
| Storage | 20 GiB General Purpose SSD |
| Network | Publicly accessible, restricted to the current CloudShell IP |
| VPC security group | `tutorial-rds-postgres-sg` |
| PostgreSQL port | `5432` |
| Practice table | `tutorial_notes` |

An RDS **DB instance** is a managed database server. AWS operates its underlying infrastructure, while you connect with normal PostgreSQL tools and manage your databases, tables, and data.

## Part 1 — Open CloudShell and record its public IP address

1. Sign in to the [AWS Management Console](https://console.aws.amazon.com/).
2. In the upper-right Region selector, choose your normal learning Region. Use this Region throughout the tutorial.
3. Choose the **CloudShell** icon in the Console navigation bar.
4. When the Bash prompt appears, confirm that the PostgreSQL client is installed:

```bash
psql --version
```

**Expected result:** A PostgreSQL client version is displayed.

5. Find the public IP address used by this CloudShell session:

```bash
curl -4 -s https://checkip.amazonaws.com
```
32.236.229.141/32

6. Copy the returned IP address into a temporary note and add `/32` to the end. For example, if the command returns `203.0.113.10`, record:

```text
203.0.113.10/32
```

The example address above is not a real address to use. A `/32` rule permits only the one IPv4 address returned by your CloudShell session.

### Checkpoint

You should have:

- A working `psql` command.
- Your current CloudShell public IP address written in `/32` format.

Keep this CloudShell session open. Its public IP can change if the environment restarts.

## Part 2 — Create a narrow security group

1. In another Console tab, search for **VPC** and open **VPC**.
2. In the left navigation pane, choose **Security groups**.
3. Choose **Create security group**.
4. For **Security group name**, enter `tutorial-rds-postgres-sg`.
5. For **Description**, enter `Allows PostgreSQL only from the tutorial CloudShell IP.`
6. For **VPC**, choose your **default VPC**.
7. Under **Inbound rules**, choose **Add rule**.
8. Configure the rule:

    - **Type:** `PostgreSQL`
    - **Protocol:** TCP
    - **Port range:** `5432`
    - **Source:** Custom
    - **Source value:** the CloudShell address recorded in Part 1, including `/32`
    - **Description:** `Tutorial CloudShell only`

9. Do not add any other inbound rule. In particular, do not use `0.0.0.0/0`.
10. Keep the default outbound rule.
11. Choose **Create security group**.

### Checkpoint

Open `tutorial-rds-postgres-sg` and confirm that its only inbound rule allows TCP port `5432` from one `/32` IPv4 address.

## Part 3 — Create the RDS for PostgreSQL instance

1. In the Console search box, enter **RDS** and open **Amazon RDS**.
2. In the left navigation pane, choose **Databases**.
3. Choose **Create database**.
4. For **Database creation method**, choose **Standard create**.
5. Under **Engine options**, choose **PostgreSQL**.
6. Keep the default PostgreSQL engine version offered by the Console.
7. Under **Templates**:

    - Choose **Free tier** if that template is displayed for your account.
    - Otherwise, choose **Dev/Test** and carefully select the small Single-AZ settings listed below.

8. Under **Availability and durability**, choose **Single DB instance**. Do not create a Multi-AZ deployment or read replica for this lab.
9. Under **Settings**, for **DB instance identifier**, enter `tutorial-postgres-db`.
10. For **Master username**, enter `tutorialadmin`.
11. Under **Credentials management**, choose **Self managed**.
12. Create a unique temporary master password, enter it in both password fields, and keep it in a secure temporary location until cleanup.

    Do not use a password from another account, put the password in a command, or save it in this tutorial file. `psql` will request it through a hidden password prompt.

13. Under **Instance configuration**, choose **Burstable classes**.
14. Choose `db.t4g.micro` if the Console marks it as Free Tier eligible. If it is unavailable, choose `db.t3.micro` when that class is marked eligible.
15. Under **Storage**:

    - Choose **General Purpose SSD**.
    - Set **Allocated storage** to `20` GiB.
    - Turn off **Storage autoscaling** for this short lab so storage cannot grow unexpectedly.

16. Under **Connectivity**, for **Compute resource**, choose **Don't connect to an EC2 compute resource**.
17. For **Network type**, choose **IPv4**.
18. For **Virtual private cloud (VPC)**, choose the same default VPC used for the security group.
19. For **DB subnet group**, keep the default subnet group for that VPC.
20. For **Public access**, choose **Yes**.

    Public accessibility provides a public endpoint, but the security group still restricts network access to the one CloudShell IP recorded earlier. This design is only for this short tutorial; production databases are normally private.

21. Under **VPC security group**, choose **Choose existing**.
22. Remove the `default` security group if the Console selected it, then select only `tutorial-rds-postgres-sg`.
23. Keep **Availability Zone** as **No preference**.
24. Keep **Database port** as `5432`.
25. Under **Database authentication**, choose **Password authentication**.
26. Expand **Additional configuration**.
27. For **Initial database name**, enter `tutorialdb`.
28. Set **Backup retention period** to `1` day.
29. Do not enable backup replication to another Region.
30. Do not export database logs to CloudWatch for this lab.
31. Leave **Enhanced Monitoring** disabled.
32. Do not enable a paid or Advanced Database Insights option. If **Database Insights — Standard** is selected automatically at no additional charge, you can leave it selected.
33. Keep **Storage encryption** enabled with the default AWS managed key.
34. Make sure **Deletion protection** is not selected so that cleanup can succeed.
35. Review the **Estimated monthly costs** panel and confirm that the selected instance is the smallest suitable option for your account.
36. Choose **Create database**.

Do not choose any banner that offers to add an EC2 connection after creation; CloudShell is the client for this lab.

### Checkpoint

The RDS **Databases** page should list `tutorial-postgres-db` with status **Creating**. Creation commonly takes several minutes. Continue only when its status changes to **Available**.

## Part 4 — Copy the database endpoint

1. On the RDS **Databases** page, open `tutorial-postgres-db`.
2. Choose the **Connectivity & security** tab if it is not already open.
3. Under **Endpoint & port**, copy the **Endpoint** into a temporary note.
4. Confirm that the port is `5432`.
5. Confirm that **Publicly accessible** is **Yes**.
6. Under **VPC security groups**, confirm that `tutorial-rds-postgres-sg` is active.

The endpoint resembles this example:

```text
tutorial-postgres-db.abc123xyz.ap-southeast-2.rds.amazonaws.com
```

Your endpoint will be different. Copy only the endpoint hostname; do not add `:5432` to it.

## Part 5 — Connect with PostgreSQL psql

1. Return to the same CloudShell session used in Part 1.
2. Replace `<your-RDS-endpoint>` in the following command with the endpoint copied from RDS:

```bash
psql "host=<your-RDS-endpoint> port=5432 dbname=tutorialdb user=tutorialadmin sslmode=require"
```

3. At the `Password for user tutorialadmin:` prompt, enter the temporary master password.

    The password does not appear while you type or paste it. This is expected.

4. Press Enter.

**Expected result:** The prompt changes to:

```text
tutorialdb=>
```

The connection information should also state that an SSL connection is being used.

### Checkpoint

Run:

```sql
SELECT current_database(), current_user;
```

**Expected result:** The result contains database `tutorialdb` and user `tutorialadmin`.

## Part 6 — Create and query a table

At the `tutorialdb=>` prompt, run each SQL statement separately.

1. Create a table:

```sql
CREATE TABLE tutorial_notes (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    note TEXT NOT NULL
);
```

**Expected result:**

```text
CREATE TABLE
```

2. Insert one row:

```sql
INSERT INTO tutorial_notes (note)
VALUES ('My first RDS PostgreSQL row.');
```

**Expected result:**

```text
INSERT 0 1
```

3. Query the table:

```sql
SELECT id, note FROM tutorial_notes;
```

**Expected result:**

```text
 id |              note
----+--------------------------------
  1 | My first RDS PostgreSQL row.
```

Spacing can differ slightly in your terminal.

## Part 7 — Run an intentional failure test

Run a query against a table that does not exist:

```sql
SELECT * FROM tutorial_table_that_does_not_exist;
```

**Expected result:** PostgreSQL returns an error similar to:

```text
ERROR:  relation "tutorial_table_that_does_not_exist" does not exist
```

This failure is intentional. It proves that PostgreSQL validates object names and reports SQL errors without ending the database session.

Run the valid query again:

```sql
SELECT id, note FROM tutorial_notes;
```

**Expected result:** The original row still appears.

## Part 8 — Reconnect and prove persistence

1. Exit `psql`:

```text
\q
```

2. Run the same connection command again, substituting your endpoint:

```bash
psql "host=<your-RDS-endpoint> port=5432 dbname=tutorialdb user=tutorialadmin sslmode=require"
```

3. Enter the temporary master password when prompted.
4. Query the table again:

```sql
SELECT id, note FROM tutorial_notes;
```

**Expected result:** The row inserted earlier still appears, even though the client disconnected and reconnected.

5. Exit `psql`:

```text
\q
```

### Checkpoint

You have proved that the PostgreSQL data is stored by the RDS DB instance and is not tied to one CloudShell client session.

## Part 9 — Clean up

The DB instance can continue generating charges until its status becomes **Deleted**. This lab intentionally skips recovery snapshots because the data is disposable.

### Delete the RDS DB instance

1. Open the Amazon RDS Console.
2. In the navigation pane, choose **Databases**.
3. Select `tutorial-postgres-db`.
4. Choose **Actions**, then **Delete**.
5. Clear **Create final snapshot** or select **No** when asked whether to create one.
6. Clear **Retain automated backups**.
7. Select the acknowledgement that the database and its data will be deleted, if shown.
8. Enter `delete me` in the confirmation box.
9. Choose **Delete**.
10. Wait until `tutorial-postgres-db` disappears from the database list. Deletion can take several minutes.

Do not create a final snapshot or retain automated backups for this disposable lab. Retained backups and snapshots can continue incurring storage charges after the DB instance is gone.

### Check for leftover snapshots and automated backups

1. In the RDS navigation pane, choose **Snapshots**.
2. Check the **Manual** snapshots list for any snapshot whose name begins with `tutorial-postgres-db`.
3. If you accidentally created one, select it and choose **Actions**, **Delete snapshot**, then confirm.
4. In the RDS navigation pane, choose **Automated backups**.
5. Open the **Retained** tab and confirm that no retained backup exists for `tutorial-postgres-db`.
6. If one exists, select it, choose **Actions**, **Delete**, and confirm.

### Delete the security group

1. Open the VPC Console.
2. Choose **Security groups**.
3. Select only `tutorial-rds-postgres-sg`.
4. Choose **Actions**, then **Delete security groups**.
5. Confirm the deletion.

If the security group is still attached to an RDS network interface, wait a few minutes after the database disappears and try again.

### Final checkpoint

Confirm that:

- RDS no longer lists `tutorial-postgres-db`.
- No manual or retained automated backup remains for the tutorial database.
- VPC no longer lists `tutorial-rds-postgres-sg`.
- No final snapshot was created.

Delete the temporary database password from wherever you stored it.

## Troubleshooting

### psql is not found in CloudShell

Run `psql --version` again in a standard CloudShell environment, not a CloudShell VPC environment. AWS CloudShell normally includes `psql`. If it is missing, restart CloudShell and check again before changing the database configuration.

### The connection times out

Confirm that the DB status is **Available**, **Publicly accessible** is **Yes**, and the endpoint and port are correct. Re-run `curl -4 -s https://checkip.amazonaws.com` in the same CloudShell session. If its address changed, edit the security group's inbound PostgreSQL rule so its source is the new IP with `/32`.

Also confirm that the RDS instance and security group use the same VPC and that the selected DB subnet group has a route to an internet gateway. The default VPC normally provides this route. If the account has no default VPC or its networking was modified, this simple public-connection lab requires a suitable VPC and DB subnet group.

### Password authentication fails

Confirm that the username is exactly `tutorialadmin` and re-enter the password stored during creation. Do not put the password directly in the connection command. If necessary, use **Modify** on the RDS instance to set a new master password, apply the change immediately, and wait until the instance returns to **Available**.

### The database name does not exist

Confirm that **Initial database name** was set to `tutorialdb`. If it was omitted, connect to the default `postgres` database instead, create the database with `CREATE DATABASE tutorialdb;`, exit, and then reconnect to `tutorialdb`.

### The DB instance cannot be deleted

Open the database, choose **Modify**, clear **Deletion protection**, and apply the change immediately. After the instance returns to **Available**, repeat the deletion steps.

## What you learned

- Amazon RDS runs and manages the PostgreSQL database server infrastructure.
- A DB instance has an endpoint and port used by standard PostgreSQL clients.
- Public accessibility alone does not permit a connection; the VPC security group must also allow the client source.
- A `/32` inbound rule restricts network access to one IPv4 address.
- `psql` can connect with TLS, create tables, insert data, query data, and display SQL errors.
- Data remains in the DB instance after the client disconnects.
- DB instances, retained backups, and manual snapshots must be removed carefully to stop related charges.

## Official AWS documentation

- [Creating an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html)
- [Creating and connecting to an RDS for PostgreSQL DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html)
- [Connect to RDS for PostgreSQL with psql](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToPostgreSQLInstance.psql.html)
- [Control access with VPC security groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html)
- [Deleting an RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html)
- [AWS CloudShell pre-installed software](https://docs.aws.amazon.com/cloudshell/latest/userguide/vm-specs.html)
- [Amazon RDS for PostgreSQL pricing](https://aws.amazon.com/rds/postgresql/pricing/)

Instructions checked on 13 September 2026. AWS Console labels and Free Tier offers can change over time.
