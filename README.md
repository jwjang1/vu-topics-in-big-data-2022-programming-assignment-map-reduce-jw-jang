# Vanderbilt - Big Data 2021 - Homework 4 Mapreduce
 
**Due Date March 31, 11:59 PM. Note that this his due date will be strictly enforced. Late penalty has been specified already in the syllabus**

**Remember** -- To Colab badge URL for all notebooks in the repository as you did in previous assignments. You can use the google chrome collab extension to help with this.
 
## Goals
 
* Complete a set of MapReduce programming tasks using MRjob python API on Colab; Read about MR job [here](https://medium.com/datable/beginners-guide-for-mapreduce-with-mrjob-in-python-dbd2e7dd0f86)
* Test the MRJob code using the local runner (-r local runner).Take a look at [0_mrjob_basics_local.ipynb](0_mrjob_basics_local.ipynb).
* Deploy and run your mapper and reducer code on AWS EMR Hadoop cluster. **Note** before you run your task on EMR its a good practice to try and run a local copy with some subset of data. You can do a local run with -r local runner. Take a look at [0_mrjob_basics_emr.ipynb](0_mrjob_basics_emr.ipynb).



# Step 1 - review the basics of MR Job

Review the information about mapreduce by going through the [Class slides](https://brightspace.vanderbilt.edu/d2l/le/content/269528/viewContent/1794302/View). In particular recall the [MapReduce Paper](mapreduce-osdi04.pdf) and [bookchapter](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267880/View). Also take a look at the  [MapReduce examples](https://github.com/vu-topics-in-big-data-2022/examples/tree/main/example-map-reduce) in the class github.

In this assignment, you will be working with MRJob. It is a high level library that makes it easy to work with map reduce tasks. Some useful links on MRJob are the following

- [uchicago](https://www.classes.cs.uchicago.edu/archive/2016/spring/12300-1/lab3.html) 
- MR Job tutorial at https://mrjob.readthedocs.io/en/latest/guides/emr-quickstart.html.
- The documentation of MRjob is available [here](https://buildmedia.readthedocs.org/media/pdf/mrjob/latest/mrjob.pdf)

For this first part of the assignment, open the 
[0_mrjob_basics_local.ipynb](0_mrjob_basics_local.ipynb) in colab and step through the cells. Try to understand what is happening. Look at the map function, and the reduce function. Look at how the map reduce task is being invoked by actually calling the python script that wraps the map and reduce functions. 

Check out the use of ```%%file mr_word_count.py``` within the file to output the content of the cell into a file. Look at the line 
```
if __name__ == '__main__': 
  MRWordFrequencyCount.run() 
```

that is used to invoke the map reduce task through the MR job library

```
!python mr_word_count.py -r local sample_data/README.md
```

Here '!' is used because we are launching a shell command from the cell. the `-r local` is the runner being passed to mrjob library. There is a `-r inline` version as well. 

# Step 2 - review the basics for using the EMR runner with the MRJob

 Amazon EMR is a cloud big data platform for processing vast amounts of data using open source tools such as [Apache Spark](https://aws.amazon.com/emr/features/spark/), [Apache Hive](https://aws.amazon.com/emr/features/hive/), [Apache HBase](https://aws.amazon.com/emr/features/hbase/), [Apache Flink](https://aws.amazon.com/blogs/big-data/use-apache-flink-on-amazon-emr/), [Apache Hudi](https://aws.amazon.com/emr/features/hudi/), and [Presto](https://aws.amazon.com/emr/features/presto/). It provides a cluster of machines. The MRJob will dispatch the work to these machines and get the output back to colab.

 Follow these steps
 1. Open the [0_mrjob_basics_emr.ipynb](0_mrjob_basics_emr.ipynb) in colab. Ensure that the url of the colab badge points to your repository.
 2. The emr version of the notebook will require you to get your AWS credentials and add them to the mrjob.conf file.

**Note** to get to the AWS CLI information you need to open and configure the [mrjob.conf](mrjob.conf) with your information
```yaml
runners: 
    emr: 
        region: xxx
        aws_access_key_id: xxx
        aws_secret_access_key: xxx
        aws_session_token: xxx
```

The image below shows where you can get this information

![images/AWSCLI.png](images/AWSCLI.png)

**Note** do not check in your credentials back to the repository.

3. Review what an ssh key is. Look at https://missing.csail.mit.edu/2019/remote-machines/. You need to configure an ssh key so that the colab can communicate with the master node of your EMR Cluster. Follow these steps
   -  create a new ssh key pair in AWS console. For this you need to be logged into the AWS academy and click on the AWS label just above the terminal where a green circle is showing. See the image below.
![images/console0.png](images/console0.png)
   - Once you reach the aws management console, search for key pairs.
   ![images/awsconsole.png](images/awsconsole.png)
   ![images/keypairs.png](images/keypairs.png)
   - Then create a key pair, call it emr and download the .pem file on your machine. Change the permission of this file to 400 ```chmod 400 emr.pem```
   - edit the mrjob.conf to add the information about the ssh key in there. Ensure that you use space properly. It is a yaml file
   ```yaml
   runners:
    emr:
        region: xxx
        aws_access_key_id: xxx
        aws_secret_access_key: xxx
        ec2_key_pair: EMR
        ec2_key_pair_file: path-on-colab-to-emr.pem 
        aws_session_token: xxx

   ```
   - Go back to colab with [0_mrjob_basics_emr.ipynb](0_mrjob_basics_emr.ipynb), and upload the emr.pem  and the mrjob.conf in the current working directory through the upload cell. See https://colab.research.google.com/notebooks/io.ipynb for more help on how to upload files to colab.
   - Ensure you run the cell that says ```!chmod 400 emr.pem```
4. Now create an s3 bucket. You did it in pandas assignment and lets say you call it vandy-bigdata. Upload a file called README.md to that bucket.
5. Create an EMR cluster. 
   - search for EMR on the aws console and open it.
   - click on create cluster.  Then select the configurations as shown in the figure below. Name your cluster and ensure auto termination policy is set. Remember when the experiment is done you must terminate the EMR cluster.
   ![images/emr0.png](images/emr0.png)
   ![images/emr1.png](images/emr1.png)
   - creation of the cluster can take some time. The state of the cluster will change from provisioning, bootstrapping to waiting. The waiting means the cluster is ready to use
   - Once the cluster is booted up, copy the cluster id. It will look something like this j-3JZUGYI3XI3MO. This is the id of your cluster.
6. Configure the security group of your master node on the cluster to allow incoming ssh connection. Find the cluster security group from the cluster status page and add a new rule that allows ssh connection from anywhere. If it was successful, you will be able to ssh into your master node from your desktop.
7. Finally, you are ready to run the [0_mrjob_basics_emr.ipynb](0_mrjob_basics_emr.ipynb) - ensure that mrjob.conf and the emr.pem has been uploaded. Then run the cell that says ```!python mr_word_count.py -r emr s3://vandy-bigdata/README.md --cloud-tmp-dir=s3://vandy-bigdata/tmp/ --cluster-id=j-3JZUGYI3XI3MO --conf-path mrjob.conf >output.emr```. Change the cluster id and s3 paths as required for your case. Follow rest of the notebook and ensure everything is working correctly. 
8. Terminate the EMR cluster.


**Note** now you will follow these steps to finish a set of excercises. It is strongly recommended that you first try out the code logic with `-r local` runner as in the 0_mrjob_basics_local notebook. Then follow the steps from above to create an emr cluster and s3 bucket and then execute the same notebook with emr. Your final code results in the notebook should include the output from the emr step showing as in the 0_mrjob_basics_emr notebook cell where we are doing ``` !cat output.emr ```. Commit the notebooks from 1_count, 2_group, 3_days and 4_joins with the correct code, execution with local and emr runners and the output.emr cat output. Submit the url of your repo in brightspace.

## Dataset Description
 
We will use four datasets as inputs during this assignment.
 
- [Tweets Dataset](data/Tweets/nashville-tweets-2019-01-28.zip)
- Baseball Dataset [Batting File](data/Baseball/Batting.csv.zip)
- Baseball Dataset [Salaries File](data/Baseball/Salaries.csv.zip)
 
Please read the readme file of each dataset first. You can find them in [data/Baseball/readme2012.txt](data/Baseball/readme2012.txt), [data/IMDB/README.md](data/IMDB/README.md) and [data/Tweets/README.md](data/Tweets/README.md)
 
**Note**  baseball and tweet data are in zip format and you have to unzip it.
 
**Note** We'll use AWS S3 as data storage in this homework, so you need to manually create an S3 bucket and upload data files into it. You did this in an earlier assignment before. 
 
# Step 3: Now finish the main Assignment Steps. That is write some code. 
 
## 1-Count the number of tweets and complete [1_count.ipynb](1_count.ipynb) -- 25 points
 
Complete the [1_count.ipynb](1_count.ipynb) notebook to count the total number of tweets in nashville-tweets-2019-01-28 data file. Note that this assumes you will use %%file magic to save the cell content of your program to 1_count.py. To test the program, please move [nashville-tweets-2019-01-28](data/Tweets/nashville-tweets-2019-01-28) to S3 bucket. 

And then run
 
```shell
python 1_count.py -r emr s3://<s3 object url of the Tweet dataset> --cloud-tmp-dir=s3://<s3 object url of tmp directory> --cluster-id=<cluster ID> --conf-path <mrjob.conf file path in colab> > 1_count.out
```
 
Capture output in your notebook
```shell
cat 1_count.out
```

now save the notebook back to Github. Remember to test with the local runner before you run with the EMR.
 
## 2-Count the screen name with the most tweets and report the count [2_group.ipynb](2_group.ipynb) -- 25 points
 
Complete the 2_group.ipynb
 
```shell
python 2_group.py -r emr s3://<s3 object url of the Tweet dataset> --cloud-tmp-dir=s3://<s3 object url of tmp directory> --cluster-id=<cluster ID> --conf-path <mrjob.conf file path in colab> > 2_group.out
```
 
Capture output in your notebook
```shell
cat 2_group.out
```
 
## 3-Count the tweets per day. [3_days.ipynb](3_days.ipynb) -- 25 points
 
Complete the 3_days.ipynb. use the same tweet dataset.

 
```shell
python 3_days.py -r emr s3://<s3 object url of the Tweet dataset> --cloud-tmp-dir=s3://<s3 object url of tmp directory> --cluster-id=<cluster ID> --conf-path <mrjob.conf file path in colab> > 3_days.out
```
 
Capture output in your notebook
```shell
cat 3_days.out
```

save your notbook back in github.
 
## 4-Join the batting and salaries data for Barry Bonds per year. [4_join.ipynb](4_join.ipynb)-- 25 points
 
- see the [Class slides](https://brightspace.vanderbilt.edu/d2l/le/content/269528/viewContent/1794302/View)about the join example and how it can be implemented using map reduce.
 
Complete the 4_join.ipynb. Note you have to upload the unizipped version of [Batting File](data/Baseball/Batting.csv.zip) and [Salaries File](data/Baseball/Salaries.csv.zip).
 
 
```shell
python 4_join.py -r emr s3://<s3 object url of the batting file> s3://<s3 object url of the salaries file> --cloud-tmp-dir=s3://<s3 object url of tmp directory> --cluster-id=<cluster ID> --conf-path <mrjob.conf file path in colab> > 4_join.out
```
 
Capture output in your notebook
```shell
cat 4_join.out
```
 
Remember to submit the url of your github repository in brightspace.
 
# References

Here are following useful references for the assignment.

* [MapReduce Paper](mapreduce-osdi04.pdf) and [bookchapter](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267880/View)
* Installing Hadoop on Linux: [part1](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267882/View), [part2](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267883/View), [part3](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267884/View)
*  MRjob - this is a library to make it easy to work with map-reduce tasks. [MapReduce examples](https://github.com/vu-topics-in-big-data-2022/examples/tree/main/example-map-reduce)
*   More MRjob examples are available at the following links: [uchicago](https://www.classes.cs.uchicago.edu/archive/2016/spring/12300-1/lab3.html) and [benjamincongdon.me](https://benjamincongdon.me/blog/2018/02/02/MapReduce-on-Python-is-better-with-MRJob-and-EMR/). Remember that you can run your MRJob tasks with the inline runner (-r inline) and local runner (-r local) to simulate the cluster and test the execution locally.
*  The documentation of MRjob is available at [here](https://buildmedia.readthedocs.org/media/pdf/mrjob/latest/mrjob.pdf).
* [MRJob Example](https://github.com/vu-topics-in-big-data-2022/examples/blob/main/example-map-reduce/mrjob/mrjob.ipynb)
* [Class slides](https://brightspace.vanderbilt.edu/d2l/le/content/269528/viewContent/1794302/View)
* Instruction about S3 from the pandas assignment
*  [Tweets Dataset](data/Tweets/nashville-tweets-2019-01-28.zip)
* Baseball Dataset [Batting File](data/Baseball/Batting.csv.zip)
*  Baseball Dataset [Salaries File](data/Baseball/Salaries.csv.zip)# Vanderbilt - Big Data 2021 - Homework 4 Mapreduce
 
**Due Date March 31, 11:59 PM. Note that this his due date will be strictly enforced. Late penalty has been specified already in the syllabus**

**Remember** -- To Colab badge URL for all notebooks in the repository as you did in previous assignments. You can use the google chrome collab extension to help with this.
 
## Goals
 
* Complete a set of MapReduce programming tasks using MRjob python API on Colab; Read about MR job [here](https://medium.com/datable/beginners-guide-for-mapreduce-with-mrjob-in-python-dbd2e7dd0f86)
* Test the MRJob code using the local runner (-r local runner).Take a look at [0_mrjob_basics_local.ipynb](0_mrjob_basics_local.ipynb).
* Deploy and run your mapper and reducer code on AWS EMR Hadoop cluster. **Note** before you run your task on EMR its a good practice to try and run a local copy with some subset of data. You can do a local run with -r local runner. Take a look at [0_mrjob_basics_emr.ipynb](0_mrjob_basics_emr.ipynb).



# Step 1 - review the basics of MR Job

Review the information about mapreduce by going through the [Class slides](https://brightspace.vanderbilt.edu/d2l/le/content/269528/viewContent/1794302/View). In particular recall the [MapReduce Paper](mapreduce-osdi04.pdf) and [bookchapter](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267880/View). Also take a look at the  [MapReduce examples](https://github.com/vu-topics-in-big-data-2022/examples/tree/main/example-map-reduce) in the class github.

In this assignment, you will be working with MRJob. It is a high level library that makes it easy to work with map reduce tasks. Some useful links on MRJob are the following

- [uchicago](https://www.classes.cs.uchicago.edu/archive/2016/spring/12300-1/lab3.html) 
- MR Job tutorial at https://mrjob.readthedocs.io/en/latest/guides/emr-quickstart.html.
- The documentation of MRjob is available [here](https://buildmedia.readthedocs.org/media/pdf/mrjob/latest/mrjob.pdf)

For this first part of the assignment, open the 
[0_mrjob_basics_local.ipynb](0_mrjob_basics_local.ipynb) in colab and step through the cells. Try to understand what is happening. Look at the map function, and the reduce function. Look at how the map reduce task is being invoked by actually calling the python script that wraps the map and reduce functions. 

Check out the use of ```%%file mr_word_count.py``` within the file to output the content of the cell into a file. Look at the line 
```
if __name__ == '__main__': 
  MRWordFrequencyCount.run() 
```

that is used to invoke the map reduce task through the MR job library

```
!python mr_word_count.py -r local sample_data/README.md
```

Here '!' is used because we are launching a shell command from the cell. the `-r local` is the runner being passed to mrjob library. There is a `-r inline` version as well. 

# Step 2 - review the basics for using the EMR runner with the MRJob

 Amazon EMR is a cloud big data platform for processing vast amounts of data using open source tools such as [Apache Spark](https://aws.amazon.com/emr/features/spark/), [Apache Hive](https://aws.amazon.com/emr/features/hive/), [Apache HBase](https://aws.amazon.com/emr/features/hbase/), [Apache Flink](https://aws.amazon.com/blogs/big-data/use-apache-flink-on-amazon-emr/), [Apache Hudi](https://aws.amazon.com/emr/features/hudi/), and [Presto](https://aws.amazon.com/emr/features/presto/). It provides a cluster of machines. The MRJob will dispatch the work to these machines and get the output back to colab.

 Follow these steps
 1. Open the [0_mrjob_basics_emr.ipynb](0_mrjob_basics_emr.ipynb) in colab. Ensure that the url of the colab badge points to your repository.
 2. The emr version of the notebook will require you to get your AWS credentials and add them to the mrjob.conf file.

**Note** to get to the AWS CLI information you need to open and configure the [mrjob.conf](mrjob.conf) with your information
```yaml
runners: 
    emr: 
        region: xxx
        aws_access_key_id: xxx
        aws_secret_access_key: xxx
        aws_session_token: xxx
```

The image below shows where you can get this information

![images/AWSCLI.png](images/AWSCLI.png)

**Note** do not check in your credentials back to the repository.

3. Review what an ssh key is. Look at https://missing.csail.mit.edu/2019/remote-machines/. You need to configure an ssh key so that the colab can communicate with the master node of your EMR Cluster. Follow these steps
   -  create a new ssh key pair in AWS console. For this you need to be logged into the AWS academy and click on the AWS label just above the terminal where a green circle is showing. See the image below.
![images/console0.png](images/console0.png)
   - Once you reach the aws management console, search for key pairs.
   ![images/awsconsole.png](images/awsconsole.png)
   ![images/keypairs.png](images/keypairs.png)
   - Then create a key pair, call it emr and download the .pem file on your machine. Change the permission of this file to 400 ```chmod 400 emr.pem```
   - edit the mrjob.conf to add the information about the ssh key in there. Ensure that you use space properly. It is a yaml file
   ```yaml
   runners:
    emr:
        region: xxx
        aws_access_key_id: xxx
        aws_secret_access_key: xxx
        ec2_key_pair: EMR
        ec2_key_pair_file: path-on-colab-to-emr.pem 
        aws_session_token: xxx

   ```
   - Go back to colab with [0_mrjob_basics_emr.ipynb](0_mrjob_basics_emr.ipynb), and upload the emr.pem  and the mrjob.conf in the current working directory through the upload cell. See https://colab.research.google.com/notebooks/io.ipynb for more help on how to upload files to colab.
   - Ensure you run the cell that says ```!chmod 400 emr.pem```
4. Now create an s3 bucket. You did it in pandas assignment and lets say you call it vandy-bigdata. Upload a file called README.md to that bucket.
5. Create an EMR cluster. 
   - search for EMR on the aws console and open it.
   - click on create cluster.  Then select the configurations as shown in the figure below. Name your cluster and ensure auto termination policy is set. Remember when the experiment is done you must terminate the EMR cluster.
   ![images/emr0.png](images/emr0.png)
   ![images/emr1.png](images/emr1.png)
   - creation of the cluster can take some time. The state of the cluster will change from provisioning, bootstrapping to waiting. The waiting means the cluster is ready to use
   - Once the cluster is booted up, copy the cluster id. It will look something like this j-3JZUGYI3XI3MO. This is the id of your cluster.
6. Configure the security group of your master node on the cluster to allow incoming ssh connection. Find the cluster security group from the cluster status page and add a new rule that allows ssh connection from anywhere. If it was successful, you will be able to ssh into your master node from your desktop.
7. Finally, you are ready to run the [0_mrjob_basics_emr.ipynb](0_mrjob_basics_emr.ipynb) - ensure that mrjob.conf and the emr.pem has been uploaded. Then run the cell that says ```!python mr_word_count.py -r emr s3://vandy-bigdata/README.md --cloud-tmp-dir=s3://vandy-bigdata/tmp/ --cluster-id=j-3JZUGYI3XI3MO --conf-path mrjob.conf >output.emr```. Change the cluster id and s3 paths as required for your case. Follow rest of the notebook and ensure everything is working correctly. 
8. Terminate the EMR cluster.


**Note** now you will follow these steps to finish a set of excercises. It is strongly recommended that you first try out the code logic with `-r local` runner as in the 0_mrjob_basics_local notebook. Then follow the steps from above to create an emr cluster and s3 bucket and then execute the same notebook with emr. Your final code results in the notebook should include the output from the emr step showing as in the 0_mrjob_basics_emr notebook cell where we are doing ``` !cat output.emr ```. Commit the notebooks from 1_count, 2_group, 3_days and 4_joins with the correct code, execution with local and emr runners and the output.emr cat output. Submit the url of your repo in brightspace.

## Dataset Description
 
We will use four datasets as inputs during this assignment.
 
- [Tweets Dataset](data/Tweets/nashville-tweets-2019-01-28.zip)
- Baseball Dataset [Batting File](data/Baseball/Batting.csv.zip)
- Baseball Dataset [Salaries File](data/Baseball/Salaries.csv.zip)
 
Please read the readme file of each dataset first. You can find them in [data/Baseball/readme2012.txt](data/Baseball/readme2012.txt), [data/IMDB/README.md](data/IMDB/README.md) and [data/Tweets/README.md](data/Tweets/README.md)
 
**Note**  baseball and tweet data are in zip format and you have to unzip it.
 
**Note** We'll use AWS S3 as data storage in this homework, so you need to manually create an S3 bucket and upload data files into it. You did this in an earlier assignment before. 
 
# Step 3: Now finish the main Assignment Steps. That is write some code. 
 
## 1-Count the number of tweets and complete [1_count.ipynb](1_count.ipynb) -- 25 points
 
Complete the [1_count.ipynb](1_count.ipynb) notebook to count the total number of tweets in nashville-tweets-2019-01-28 data file. Note that this assumes you will use %%file magic to save the cell content of your program to 1_count.py. To test the program, please move [nashville-tweets-2019-01-28](data/Tweets/nashville-tweets-2019-01-28) to S3 bucket. 

And then run
 
```shell
python 1_count.py -r emr s3://<s3 object url of the Tweet dataset> --cloud-tmp-dir=s3://<s3 object url of tmp directory> --cluster-id=<cluster ID> --conf-path <mrjob.conf file path in colab> > 1_count.out
```
 
Capture output in your notebook
```shell
cat 1_count.out
```

now save the notebook back to Github. Remember to test with the local runner before you run with the EMR.
 
## 2-Count the screen name with the most tweets and report the count [2_group.ipynb](2_group.ipynb) -- 25 points
 
Complete the 2_group.ipynb
 
```shell
python 2_group.py -r emr s3://<s3 object url of the Tweet dataset> --cloud-tmp-dir=s3://<s3 object url of tmp directory> --cluster-id=<cluster ID> --conf-path <mrjob.conf file path in colab> > 2_group.out
```
 
Capture output in your notebook
```shell
cat 2_group.out
```
 
## 3-Count the tweets per day. [3_days.ipynb](3_days.ipynb) -- 25 points
 
Complete the 3_days.ipynb. use the same tweet dataset.

 
```shell
python 3_days.py -r emr s3://<s3 object url of the Tweet dataset> --cloud-tmp-dir=s3://<s3 object url of tmp directory> --cluster-id=<cluster ID> --conf-path <mrjob.conf file path in colab> > 3_days.out
```
 
Capture output in your notebook
```shell
cat 3_days.out
```

save your notbook back in github.
 
## 4-Join the batting and salaries data for Barry Bonds per year. [4_join.ipynb](4_join.ipynb)-- 25 points
 
- see the [Class slides](https://brightspace.vanderbilt.edu/d2l/le/content/269528/viewContent/1794302/View)about the join example and how it can be implemented using map reduce.
 
Complete the 4_join.ipynb. Note you have to upload the unizipped version of [Batting File](data/Baseball/Batting.csv.zip) and [Salaries File](data/Baseball/Salaries.csv.zip).
 
 
```shell
python 4_join.py -r emr s3://<s3 object url of the batting file> s3://<s3 object url of the salaries file> --cloud-tmp-dir=s3://<s3 object url of tmp directory> --cluster-id=<cluster ID> --conf-path <mrjob.conf file path in colab> > 4_join.out
```
 
Capture output in your notebook
```shell
cat 4_join.out
```
 
Remember to submit the url of your github repository in brightspace.
 
# References

Here are following useful references for the assignment.

* [MapReduce Paper](mapreduce-osdi04.pdf) and [bookchapter](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267880/View)
* Installing Hadoop on Linux: [part1](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267882/View), [part2](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267883/View), [part3](https://brightspace.vanderbilt.edu/d2l/le/content/342996/viewContent/2267884/View)
*  MRjob - this is a library to make it easy to work with map-reduce tasks. [MapReduce examples](https://github.com/vu-topics-in-big-data-2022/examples/tree/main/example-map-reduce)
*   More MRjob examples are available at the following links: [uchicago](https://www.classes.cs.uchicago.edu/archive/2016/spring/12300-1/lab3.html) and [benjamincongdon.me](https://benjamincongdon.me/blog/2018/02/02/MapReduce-on-Python-is-better-with-MRJob-and-EMR/). Remember that you can run your MRJob tasks with the inline runner (-r inline) and local runner (-r local) to simulate the cluster and test the execution locally.
*  The documentation of MRjob is available at [here](https://buildmedia.readthedocs.org/media/pdf/mrjob/latest/mrjob.pdf).
* [MRJob Example](https://github.com/vu-topics-in-big-data-2022/examples/blob/main/example-map-reduce/mrjob/mrjob.ipynb)
* [Class slides](https://brightspace.vanderbilt.edu/d2l/le/content/269528/viewContent/1794302/View)
* Instruction about S3 from the pandas assignment
*  [Tweets Dataset](data/Tweets/nashville-tweets-2019-01-28.zip)
* Baseball Dataset [Batting File](data/Baseball/Batting.csv.zip)
*  Baseball Dataset [Salaries File](data/Baseball/Salaries.csv.zip)
