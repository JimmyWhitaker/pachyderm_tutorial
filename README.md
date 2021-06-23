# Pachyderm Tutorial
> This is a repo for a Pachyderm tutorial


```python
%load_ext autoreload
%autoreload 2
```

This file will become your README and also the index of your documentation.

## Install

1. Create a free Pachyderm cluster on [Pachyderm Hub](https://hub.pachyderm.com/).

2. Connect to JupyterHub in the cluster

3. Clone this repo 

`pip install pachyderm_tutorial`

## Pachyderm File System Basics

Create a Pachyderm data repository called `data`

```python
!pachctl version
```

    COMPONENT           VERSION             
    pachctl             1.13.0              
    pachd               1.12.5              


```python
!pachctl create repo data
```

```python
!pachctl list repo
```

    NAME CREATED      SIZE (MASTER) ACCESS LEVEL 
    data 1 second ago 0B            OWNER         


When we list our repos, we can see that we have an empty data repository, with no data in it. So let's add some data.

```python
%%writefile iris.csv
5.1,3.5,1.4,0.2,Iris-setosa
4.9,3.0,1.4,0.2,Iris-setosa
4.7,3.2,1.3,0.2,Iris-setosa
4.6,3.1,1.5,0.2,Iris-setosa
7.0,3.2,4.7,1.4,Iris-versicolor
6.4,3.2,4.5,1.5,Iris-versicolor
6.9,3.1,4.9,1.5,Iris-versicolor
5.5,2.3,4.0,1.3,Iris-versicolor
6.3,3.3,6.0,2.5,Iris-virginica
5.8,2.7,5.1,1.9,Iris-virginica
7.1,3.0,5.9,2.1,Iris-virginica
6.3,2.9,5.6,1.8,Iris-virginica
```

    Writing iris.csv


Data repositories in Pachyderm automatically track versions of the data placed in  them. Similar to Git, we organize our data via branches, so we will push our data to the `master` branch of our `data` repository.

```python
!pachctl put file data@master -f iris.csv
```

    iris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s


We can look at the data that's been uploaded to our `data` repository, by listing the files on the master branch.

```python
!pachctl list file data@master
```

    NAME      TYPE SIZE 
    /iris.csv file 364B 


Similarly, if we want to delete our file we can do that as well. 

```python
!pachctl delete file data@master:/iris.csv
```

```python
!pachctl list file data@master
```

    NAME TYPE SIZE 


Now let's add it back again. 

```python
!pachctl put file data@master -f iris.csv
```

    iris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s
    [1A[Jiris.csv 364.00b / 364.00 b [======================================] 0s 0.00 b/s


Now we can list all of the commits that have been made to our repository to see the history of the `master` branch.

```python
!pachctl list commit data
```

    REPO BRANCH COMMIT                           FINISHED       SIZE PROGRESS DESCRIPTION
    data master be040bd068fb4e67ae59cd2f40bb2a0c 16 minutes ago 364B -         
    data master 44e71eeb040e4453a13014c22c9a80ed 17 minutes ago 0B   -         
    data master 4670ab8ce78e4306a5789144f30b4afd 20 minutes ago 364B -         


Pachyderm keeps a record of all the changes that happen to the `data` repository. This way if we ever want to revert to a previous version of our dataset, we can do it. 

For example, if we wanted to change move the head of our `master` branch to the first commit, we could run the following command (note the commit hashes will be different if you are running this yourself): 

```python
!pachctl create branch data@master --head 4670ab8ce78e4306a5789144f30b4afd
```

```python
!pachctl list branch data
```

    BRANCH HEAD                             TRIGGER 
    master 4670ab8ce78e4306a5789144f30b4afd -       


We can also use [Ancestry Syntax](https://docs.pachyderm.com/latest/concepts/data-concepts/history/#ancestry-syntax) to traverse commits. `^` for the parent of the commit or we can reference the commits in numerical order using `.n`, where `n` is the commit number. 

```python
# Reset the head of our master branch
!pachctl create branch data@master --head be040bd068fb4e67ae59cd2f40bb2a0c
```

```python
# Previous commit to the head of the master branch
!pachctl list commit data@master^
```

    REPO BRANCH COMMIT                           FINISHED       SIZE PROGRESS DESCRIPTION
    data master 44e71eeb040e4453a13014c22c9a80ed 17 minutes ago 0B   -         
    data master 4670ab8ce78e4306a5789144f30b4afd 21 minutes ago 364B -         


```python
# First commit to the master branch
!pachctl list commit data@master.1
```

    REPO BRANCH COMMIT                           FINISHED       SIZE PROGRESS DESCRIPTION
    data master 4670ab8ce78e4306a5789144f30b4afd 21 minutes ago 364B -         


```python
!pachctl list commit data@master.-1
```

    REPO BRANCH COMMIT                           FINISHED       SIZE PROGRESS DESCRIPTION
    data master 44e71eeb040e4453a13014c22c9a80ed 17 minutes ago 0B   -         
    data master 4670ab8ce78e4306a5789144f30b4afd 21 minutes ago 364B -         


```python
!pachctl list branch data
```

    BRANCH HEAD                             TRIGGER 
    master be040bd068fb4e67ae59cd2f40bb2a0c -       


## Pachyderm Pipeline System Basics

Pachyderm also lets you create code pipelines that can be triggered by your data. These pipelines connect to your data repositories, ensuring that they run anytime your data is updated.

Here is an example of a Pachyderm pipeline. 

```python
%%writefile count.yaml
pipeline:
  name: count
description: Count the number of lines in a csv file
input:
  pfs:
    glob: /
    repo: data
transform:
  cmd: ['/bin/sh']
  stdin: ['wc -l /pfs/data/iris.csv > /pfs/out/line_count.txt']
  image: alpine:3.14.0
```

    Writing count.yaml


When our pipeline runs, it will map the data from our `data` repository into the container that's running our pipeline. It will also automatically create a new data repository to hold and version our output(s). 

We can submit our pipeline to Pachyderm by using the `create pipeline` command.

```python
!pachctl create pipeline -f count.yaml
```

If we list our pipelines, we can see the  status of them.

```python
!pachctl list pipeline
```

    NAME  VERSION INPUT  CREATED        STATE / LAST JOB  DESCRIPTION                             
    count 1       data:/ 15 seconds ago [32mrunning[0m / [32msuccess[0m Count the number of lines in a csv file 


It looks like our pipeline failed, so let's inspect it and see why.

```python
!pachctl list job --pipeline=count
```

    ID                               PIPELINE STARTED       DURATION  RESTART PROGRESS  DL   UL  STATE   
    9bd27e46d1b64bd0bde16acefa5dff15 count    4 seconds ago 2 seconds 0       1 + 0 / 1 364B 22B [32msuccess[0m 


We can inspect the logs for the pipeline to see what went wrong. (Note there are 3 [tries](https://docs.pachyderm.com/latest/reference/pipeline_spec/#datum-tries-optional) here.)

```python
!pachctl list file count@master
```

    NAME            TYPE SIZE 
    /line_count.txt file 22B  


```python
!pachctl get file count@master:/line_count.txt -o ./line_count.txt
```

    ./line_count.txt 0.00b / 22.00 b [---------------------------------] 0s 0.00 b/s
    [1A[J./line_count.txt 22.00b / 22.00 b [================================] 0s 0.00 b/s
    [1A[J./line_count.txt 22.00b / 22.00 b [================================] 0s 0.00 b/s
    [1A[J./line_count.txt 22.00b / 22.00 b [================================] 0s 0.00 b/s


```python
cat line_count.txt
```

    12 /pfs/data/iris.csv


Clear all our data out for the next section

```python
## Uncomment and run if continuing on
# !pachctl delete pipeline --all
# !pachctl delete repo --all
```

## Python-Pachyderm

Create a client to connect to the Pachyderm clutser.

```python
print(python_pachyderm.__version__)
```

    6.2.0


```python
client = python_pachyderm.Client.new_from_config()
```

```python
client.create_repo('data')
```




    



```python
with client.commit("data", "master") as commit:
    client.put_file_bytes(commit, "data.txt", b"DATA")
```

```python
client.list_repo()
```




    [repo {
      name: "data"
    }
    created {
      seconds: 1624489726
      nanos: 662871220
    }
    size_bytes: 4
    auth_info {
      access_level: OWNER
    }
    branches {
      repo {
        name: "data"
      }
      name: "master"
    }
    ]



```python
file_list = client.list_file(('data','master'), '/', include_contents=True)

for f in file_list: 
    print(f)
```

    file {
      commit {
        repo {
          name: "data"
        }
        id: "3309835f727c4e4aa760ebf26421cdb8"
      }
      path: "/data.txt"
    }
    file_type: FILE
    size_bytes: 4
    hash: "\330Ukb\227\364\273^7gc\022\025\254\201\004\000\026\331\314G\013\003^)\006\030\304]\004\305\025"
    objects {
      hash: "4ba7d4149c32f5ccc6e54190beef0f503d1e637249baa9e4b123f5aa5c89506f299c10a7e32ab1e4bae30ed32df848f87d9b03a640320b0ca758c5ee56cb2db4"
    }
    committed {
      seconds: 1624489733
      nanos: 99330690
    }
    

