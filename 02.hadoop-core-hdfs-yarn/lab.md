# Big Data Ecosystem

## Lab 2: HDFS and YARN interaction

### Copy files to HDFS

Using the official [HDFS DFS Commands Guide](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html):

1. Create a directory named after your `$USER` in your group HDFS directory. E.g. `/education/ece_2023_spring_app_1/$USER`

answer : (replace a.nachid-ece with your username)
   ```
   hdfs dfs -mkdir -p /education/ece_2023_spring_app_1/a.nachid-ece
   ```
2. Create a subdirectory `lab2` in the directory created in 1.

answer : (replace a.nachid-ece with your username)
   ```
   hdfs dfs -mkdir -p /education/ece_2023_spring_app_1/a.nachid-ece/lab2
   ```
3. Create a file named `sentence1.txt` on the local file system and write a sentence inside that file

answer :
   ```
   hdfs dfs -mkdir sentence1.txt
   ```
   Then
   ```
   echo "write smt" > sentence1.txt
   ```
4. Copy the file to your `lab2` HDFS directory

answer : (replace a.nachid-ece with your username)
   ```
   hdfs dfs -put sentence1.txt /education/ece_2023_spring_app_1/a.nachid-ece/lab2
   ```
5. Get information about the blocks composing your file, their replica and locations using `hadoop fsck`

answer : (replace a.nachid-ece with your username)
   ```
   hdfs fsck /education/ece_2023_spring_app_1/a.nachid-ece/lab2/sentence1.txt -files -blocks -locations
   ```

### Submit an application in YARN

To test YARN, we will run the Pi example from the MapReduce example JAR. This is a MapReduce program that computes Pi using the [Quasi-Monte Carlo method](https://en.wikipedia.org/wiki/Quasi-Monte_Carlo_method) (1st argument = number of mappers, 2nd argument = number of samples).

1. Run the command:
   ```
   yarn jar /usr/hdp/3.1.0.0-78/hadoop-mapreduce/hadoop-mapreduce-examples-3.1.1.3.1.0.0-78.jar pi 6 100000000
   ```
2. Wait until you find the result
3. Run the command `yarn app -list` that shows your apps running in YARN

### Setting up password-less SSH

Follow tutorial: [How to Setup Passwordless SSH Login](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/)

### Accessing Hadoop Web UIs using Kerberos

To see the Hadoop Web UIs, you will need to configure your computer. The Kerberos protocol is used to authenticate.

The configuration depends on your OS:

- On Linux and MacOS

  - Nothing to install
  - Create the file `/etc/krb5.conf` on your computer :
  
  answer :
     ```
     sudo vim /etc/krb5.conf
     ```
  Then do ':%d' if the file is not empty to erase it's content
  
  Then press 'i' et paste the following content :

    ```ini
    [libdefaults]
     default_ccache_name = FILE:/tmp/krb5cc_%{uid}
     default_realm = AU.ADALTAS.CLOUD
     dns_lookup_realm = false
     dns_lookup_kdc = true
     rdns = false
     ticket_lifetime = 24h
     forwardable = true
     udp_preference_limit = 0
    [realms]
     AU.ADALTAS.CLOUD = {
      kdc = ipa1.au.adaltas.cloud:88
      master_kdc = ipa1.au.adaltas.cloud:88
      admin_server = ipa1.au.adaltas.cloud:749
      default_domain = au.adaltas.cloud
     }
    [domain_realm]
     .au.adaltas.cloud = AU.ADALTAS.CLOUD
     au.adaltas.cloud = AU.ADALTAS.CLOUD
     ipa1.au.adaltas.cloud = AU.ADALTAS.CLOUD
    ```
  Then do ':wq!' to save it

  - After that you can get a Kerberos ticket using `kinit a.nachid-ece` (replace a.nachid-ece with your username)
  - Check your ticket with `klist`
  - Add the following properties in your Firefox `about:config` page: (Use the STRING option)
    ```ini
    network.negotiate-auth.delegation-uris = .au.adaltas.cloud
    network.negotiate-auth.trusted-uris = .au.adaltas.cloud
    ```

- On Windows, the installation is more tricky. Follow this article: [Kerberos and Spnego authentication on Windows with Firefox](https://www.adaltas.com/en/2019/11/04/windows-krb5-client-spnego/).

Once that your computer and your Firefox browser are configured and that you have a Kerberos ticket:

1. Open the page http://yarn-rm-1.au.adaltas.cloud:8088/ui2/#/yarn-apps/apps
2. Run the Pi example once again :
   ```
   yarn jar /usr/hdp/3.1.0.0-78/hadoop-mapreduce/hadoop-mapreduce-examples-3.1.1.3.1.0.0-78.jar pi 6 100000000
   ```
3. Locate your application in the UI and check resource usage
4. Rerun the example several times and see the locations of the containers changing

### WordCount example application

The WordCount example is also located in the MapReduce example JAR. It takes several arguments:

- 1 or more input directory
- 1 output directory

1. Look at the content of the input directory we will use: `/education/ece_2023_spring_app_1/resources/lab2`

answer :
   ```
   hdfs dfs -ls /education/ece_2023_spring_app_1/resources/lab4/
   ```
2. Run the command: (replace a.nachid-ece with your username)
   ```bash
   yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples-3.1.1.3.1.0.0-78.jar \
    wordcount /education/ece_2023_spring_app_1/resources/lab4 \
    /education/ece_2023_spring_app_1/a.nachid-ece/lab2/output-moby-dick
   ```
3. Check out the output directory

answer : (replace a.nachid-ece with your username)
   ```
   hdfs dfs -cat /education/ece_2023_spring_app_1/a.nachid-ece/lab2/output-moby-dick/part-r-00000
   ```
