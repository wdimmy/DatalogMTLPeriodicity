# Materialisation-based Reasoning in DatalogMTL with Bounded Intervals

This repository contains code and other related resources of our paper "Materialisation-based Reasoning in DatalogMTL with Bounded Intervals"
published in AAAI 2023 (To appear).

****
If you find our paper and resources useful, please kindly leave a star and cite our papers. Thanks!

```bibtex
@inproceedings{aaai2023,
  title={Materialisation-based Reasoning in DatalogMTL with Bounded Intervals},
  author={Przemysław A. Wał˛ega , Michał Zawidzki , Dingmin Wang, Bernardo Cuenca Grau},
  booktitle={Proceedings of the AAAI Conference on Artificial Intelligence},
  year={2023}
}
```
****


<span id='overview'/>

### Overview:
* <a href='#introduction'>1. Introduction</a>
* <a href='#data'>2. Automatic Temporal Data Generation </a>
    * <a href='#lubm'>2.1. LUBM</a>
      * <a href="#downloadlubm">2.1.1 Download LUBM generator</a>
      * <a href="#datalog">2.1.2 Obtain datalog facts</a>
      * <a href="#datalogmtl">2.1.3 Add intervals</a>
      
    * <a href='#itemporal'>2.2. iTemporal</a>

* <a href='#example'>3. Examples of Getting the Periodic Structure </a>

* <a href='#factentailment'>4. Fact Entailment Based on the Periodic Structure </a>

****

<span id='introduction'/>

#### 1. Introduction: <a href='#overview'>[Back to Top]</a>
DatalogMTL is a powerful extension of Datalog with opera- tors from metric temporal logic (MTL),
which has received significant attention in recent years. In this paper, we inves- tigate materialisation-based 
reasoning (a.k.a. forward chain- ing) in the context of DatalogMTL programs and datasets with bounded intervals, 
where partial representations of the canonical model are obtained through successive rounds of rule applications. 
Although materialisation does not naturally terminate in this setting, it is known that the structure of canonical 
models is ultimately periodic. Our first contribution in this paper is a detailed analysis of the periodic structure 
of canonical models; in particular, we formulate saturation con- ditions whose satisfaction by a partial materialisation 
implies an ability to recover the full canonical model via unfolding; this allows us to compute the actual periods describing 
the repeating parts of the canonical model as well as to estab- lish concrete bounds on the number of rounds of rule applications 
required to achieve saturation. Based on these the- oretical results, we propose a practical reasoning algorithm where saturation 
can be efficiently detected as materialisa- tion progresses, and where the relevant periods used to eval- uate entailment of queries 
via unfolding are efficiently com- puted. We have implemented our algorithm and our experi- ments suggest that our approach
is both scalable and robust.

****

<span id="data"/>

#### 2. Automatic Temporal Data Generation </a>

<span id="lubm"/>

##### 2.1 Lehigh University Benchmark (LUBM)

<span id="downloadlubm"/>

######  2.1.1 Download the LUBM data generator

You can download the data generator (UBA) from **SWAT Projects - Lehigh University Benchmark (LUBM)** [website](http://swat.cse.lehigh.edu/projects/lubm/). In particular,
we used [UBA1.7](http://swat.cse.lehigh.edu/projects/lubm/uba1.7.zip).

After downloading the  UBA1.7 package, you need to add package's path to CLASSPATH. For examole,

```shell
export CLASSPATH="$CLASSPATH:your package path"
```

<span id="datalog"/>

###### 2.1.2 Generate the owl files
```
==================
USAGES
==================

command:
   edu.lehigh.swat.bench.uba.Generator
      	[-univ <univ_num>]
	[-index <starting_index>]
	[-seed <seed>]
	[-daml]
	-onto <ontology_url>

options:
   -univ number of universities to generate; 1 by default
   -index starting index of the universities; 0 by default
   -seed seed used for random data generation; 0 by default
   -daml generate DAML+OIL data; OWL data by default
   -onto url of the univ-bench ontology
```

We found some naming and storage issues when using the above command provided 
by the official documentation. To provide a more user-friendly way, we 
wrote a script which can be directly used to generate required owl files
by passing some simple arguments. An example is shown as follows,

```python
from meteor_reasoner.datagenerator import generate_owl

univ_nume = 1 # input the number of universities you want to generate
dir_name = "./data" # input the directory path used for the generated owl files.

generate_owl.generate(univ_nume, dir_name)

```
In  **./data**, you should obtain a serial of owl files like below,
```
University0_0.owl 
University0_12.owl  
University0_1.owl
University0_4.owl
.....
```

Then, we need to convert the owl files to datalog-like facts. We also prepare
a script that can be directly used to do the conversion. 
```python
from meteor_reasoner.datagenerator import generate_datalog

owl_path = "owl_data" # input the dir_path where owl files locate
out_dir = "./output" # input the path for the converted datalog triplets

generate_datalog.extract_triplet(owl_path, out_dir)
```
In **./output**, you should see a **./output/owl_data**  containing data
in the form of
```
UndergraduateStudent(ID0)
undergraduateDegreeFrom(ID1,ID2)
takesCourse(ID3,ID4)
undergraduateDegreeFrom(ID5,ID6)
UndergraduateStudent(ID7)
name(ID8,ID9)
......
```
and **./output/statistics.txt**  containing the statistics information
about the converted datalog-like data in the form of
```
worksFor:540
ResearchGroup:224
....
AssistantProfessor:146
subOrganizationOf:239
headOf:15
FullProfessor:125
The number of unique entities:18092
The number of triplets:8604
```
<span id="datalogmtl"/>

###### Add intervals

Up to now, we only construct the atemporal data, so the final step will be adding temporal information
(intervals) to these atemporal data.  We create a script to automatically add intervals to the atemporal
data. The required arguments are shown as follows
```
  --datalog_file_path DATALOG_FILE_PATH
  --factnum FACTNUM
  --intervalnum INTERVALNUM
  --unit UNIT
  --punctual            specify whether we only add punctual intervals
  --openbracket         specify whether we need to add open brackets
  --min_val MIN_VAL
  --max_val MAX_VAL
```

To be more specific, assuming that we have a datalog-like dataset in **datalog/datalog_data.txt**,
if we want to create a dataset containing 10000 facts and each facts has at most 2 intervals,
we can run he following command
```shell

python add_intervals.py --datalog_file_path datalog/datalog_data.txt --factnum 10000 --intervalnum 2

```

In the **datalog/10000.txt**, there should be 10000 facts, each of which in the form P(a,b)@\varrho, and 
a sample of facts are shown as follows,
```
UndergraduateStudent(ID0)@[1,18]
undergraduateDegreeFrom(ID1,ID2)@[7,18]
takesCourse(ID3,ID4)@[12,46]
undergraduateDegreeFrom(ID5,ID6)@[21,24]
UndergraduateStudent(ID7)@[3,10]
name(ID8,ID9)@[5,22]
......
```
<span id="itemporal"/>

##### itemporal 

For the dataset generation based on the iItemporal platform, we refer readers to the 
[official github repository](https://github.com/kglab-tuwien/iTemporal), where a nice web-based  
interface and an easy-to-configure file have been provided for the data generation. A more technical
details about iTemporal can also be found in their [ICDE 2022](https://ieeexplore.ieee.org/document/9835220). 


<span id="example"/>

#### 3. Examples of Getting the Periodic Structure</a>

Assume we have an example dataset (/demos/case_0_dataset.txt)
```
P@0
Q@1.5
```

and an example program (/demos/case_0_program.txt).
```
Boxplus[0,1]P:-P
Boxminus[1,1]Q:-Q
```
Obviously, the above program  is **NOT** finitely materialisable for the dataset. To obtain the periodic structure of canonical models, we can run the following command:
```shell 
python run.py --datapath ./demos/case_0_dataset.txt --rulepath ./demos/case_0_program.txt
```

The output should be like this:
```
The w is: 2
The maximum number: 1.5
The minimum number: 0
left period: [-3.5,-2.5)
Q ['[-3.5,-3.5]']
right period: (4.0,4.5]
P ['(4.0,4.5]']
```
--------------------------------------------------------------------------------

Assume we have an example dataset (/demos/case_1_dataset.txt)
```
P(a)@0
R(a,b)@0
R(b,c)@0
R(c,a)@0
```

and an example program (/demos/case_1_program.txt).
```
R(X,Y):- Diamondminus[1,1]R(X,Y)
Q(Y):- P(X), R(X,Y)
P(X):- Diamondminus[1,1]Q(X)
```
Obviously, the above program  is **NOT** finitely materialisable for the dataset. To obtain the periodic structure of canonical models, we can run the following command:
```shell 
python run.py --datapath ./demos/case_1_dataset.txt --rulepath ./demos/case_1_program.txt
```

The output should be like this:
```
The w is:2
The maximum number in the database:0
The minimum number in the database:0
left period: [-2,0)
[]
right period: (3,6]
P(a) ['[6,6]']
R(a,b) ['[4,4]', '[5,5]', '[6,6]']
R(b,c) ['[4,4]', '[5,5]', '[6,6]']
R(c,a) ['[4,4]', '[5,5]', '[6,6]']
Q(b) ['[6,6]']
P(b) ['[4,4]']
Q(c) ['[4,4]']
P(c) ['[5,5]']
Q(a) ['[5,5]']
```

**Note that** the left period is empty, so it means that there are no facts entailed in the interval (-inf,0).
If the right period "(t_1, t_2]: []", it will mean that  there are no facts entailed in the interval [t2,+inf).

--------------------------------------------------------------------------------


Assume we have an example dataset (/demos/case_15_dataset.txt)
```
P@10
Q@12

```

and an example program (/demos/case_15_program.txt).
```
F:- Diamondminus[0,2]P, Q
```
Obviously, the above program  is **finitely materialisable** for the dataset. Hence, we can obtain a dataset 
representing the canonical models, we can run the following command:
```shell 
python run.py --datapath ./demos/case_15_dataset.txt --rulepath ./demos/case_15_program.txt
```

The output should be like this:
```
The w is:4
The maximum number in the database:12
The minimum number in the database:10
This program is finitely materialisable for this dataset.
P@[10,10]
Q@[12,12]
F@[12,12]
```
--------------------------------------------------------------------------------


<span id="factentailment"/>

#### 4. Fact Entailment Based on the Periodic Structure</a>

```
python run.py --datapath ./demos/case_0_dataset.txt --rulepath ./demos/case_0_program.txt --fact "P@100.5"
```

The output should be like this
```
The w is:2
The maximum number in the database:1.5
The minimum number in the database:0
left period: [-3.5,-2.5)
Q ['[-3.5,-3.5]']
right period: (4.0,4.5]
P ['(4.0,4.5]']
Entailment: True
```
