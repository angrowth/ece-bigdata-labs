# Big Data Ecosystem

## Lab 3: The MapReduce Framework

### Run a Python MapReduce word count job using Hadoop Streaming

We will run a MapReduce job that does a word count on text files. It will return the list of words found it the input and the occurence of each one:

```
unclouded       1
uncomfortable   3
uncommonly      6
uncompromised   1
```

1. Clone the repo [ece-bigdata-2023-spring](https://github.com/adaltas/ece-bigdata-2023-spring.git) in your home on the edge node:
   ```bash
   git clone https://github.com/adaltas/ece-bigdata-2023-spring.git
   ```
2. Go to the `modules/03.the-mapreduce-framework/lab-resources/code` directory:
   ```bash
   cd ece-bigdata-2023-spring/modules/03.the-mapreduce-framework/lab-resources/code
   ```
3. Take a look at the `word_count/mapper.py` and `word_count/reducer.py` files.

answer :
   ```
   vim word_count/mapper.py    (use :q to quit vim after)
   or
   cat word_count/mapper.py
   ```
   ```
   vim word_count/reducer.py    (use :q to quit vim after)
   or
   cat word_count/reducer.py
   ```
4. Take a look at the input we will use for the MapReduce in HDFS at `/education/ece_2023_spring_app_1/resources/lab3/mapred-input`

answer :
   ```
   hdfs dfs -cat /education/ece_2023_spring_app_1/resources/lab3/mapred-input/moby-dick-aa
   ```
  
5. The following command will run a MapReduce with 2 reducers, so it will produce 2 output files: (replace a.nachid-ece with your username)

   ```bash
   mapred streaming -D mapreduce.job.reduces=2 \
     -files word_count/mapper.py,word_count/reducer.py \
     -input /education/ece_2023_spring_app_1/resources/lab3/mapred-input \
     -output /education/ece_2023_spring_app_1/a.nachid-ece/lab3/word-count \
     -mapper "python3 mapper.py" -reducer "python3 reducer.py"
   ```
6. Check the output

answer : (replace a.nachid-ece with your username)
   ```
   hdfs dfs -cat /education/ece_2023_spring_app_1/a.nachid-ece/lab3/word-count/part-00000
   ```

### Design a MapReduce job to get the most frequent word

Now, we want to get the **most frequent word** of the input by using the output of our previous `word_count` job.

Design a MapReduce job by defining:

- The output produced by the mapper
- The computation performed by the reducer

**Do not cheat:** You cannot control the number of reduce tasks that will run (their can be more than 1 reduce in 1 reducer container).

### Write the most_frequent MapReduce job

1. Implement the `most_frequent` MapReduce job in Python. Use the `word_count` mapper and reducer as inspiration.

answer :
   ```
   cd ece-bigdata-2023-spring/modules/03.the-mapreduce-framework/lab-resources/code
   ```
   ```
   mkdir most_frequent
   ```
   ```
   cd most_frequent/
   ```
   ```
   vim mapper.py
   ```
   Then paste the code bellow :
   ```
import sys

def read_kv_array(text):
    key_values = []

    for line in text:
        key, value = line.split('\t')
        key_values.append({'key': key, 'value': value})

    return key_values

def write_kv_dict(key_values):
    for k, v in key_values.items():
        print(f'{k}\t{v}')

if _name_ == '_main_':
    write_kv_dict(read_kv_array(sys.stdin))
   ```
   Then press ':wq!' to save
   
   Then
   ```
   vim reducer.py
   ```
   Then paste the code bellow :
   ```
import sys

def read_kv_array(text):
    key_values = []

    for line in text:
        key, value = line.split('\t')
        key_values.append({'key': key, 'value': int(value)})

    return key_values

def write_kv_dict(key_values):
    for k, v in key_values.items():
        print(f'{k}\t{v}')

def most_frequent(words):
    sorted_words = sorted(words, key=lambda x: x['value'], reverse=True)
    return sorted_words[:1]

if _name_ == '_main_':
    write_kv_dict(most_frequent(read_kv_array(sys.stdin)))
   ```
   Then press ':wq!' to save
   
2. Run your job. (replace a.nachid-ece with your username)
   ```sh
   mapred streaming -D stream.non.zero.exit.is.failure=false -files most_frequent/mapper.py,most_frequent/reducer.py -input /education/ece_2023_spring_app_1/a.nachid-ece/lab3/word-count -output /education/ece_2023_spring_app_1/a.nachid-ece/lab3/most-frequent -mapper "python3 mapper.py" -reducer "python3 reducer.py"
   ```
