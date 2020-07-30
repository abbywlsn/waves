---
layout: post
title: "Systematics Normalization"
date: 2020-06-03
author: Abigail Wilson
---


# Systematics Normalization
I collaborated on this project with my mentors, Emily Dolson and Kate Skocelas.


## Goal: 
The purpose of this project is to develop a way to compare phylogenetic trees of different sizes and characteristics in a standardized way. 

#### Motivation: 
Within Empirical as it is now, there is not a good way to compare phylogenetic trees with one another. Different generational sizes and characteristics make it hard to obtain statistically significant data in terms of comparison. This project aims to solve this. 

+++++ "Different generational sizes and characteristics make it hard to obtain statistically significant data in terms of comparison." -- I would reword this for clarity. You don't actually have trees as subjects in the sentence, and the trees being compared are divided in the sentence from the word comparison at the end. Maybe something like "Different tree generational sizes and characteristics make it difficult to compare them in a way that produces statistically significant data."

#### Steps: 
 1. Creation of a **null model** of a phylogenetic tree 

 2. Creation of a tree that **mutates** and diverges in a non-random way
 3. Creation of a tree which accounts for **pressure for diversity** and **mutation**
 4. Comparing trees from steps 2 and 3 with each other and the null model 
 
 +++++ Double check with Emily, but I think you'd want this to be active tense, like "create a phylogenetic tree null model... compare trees from..."

 -------------------------------------------

## Background and Definitions 

### **Phylogenetic Trees**

A phylogenetic tree is a commonly used diagram in biology that shows evolutionary relationships between organisms. The tree begins with a single population and branches when a portion of the population mutates. Because of this, phylogenetic trees are a great way to show how various species have evolved from a single individual or group.

_This is a diagram of a basic phylogenetic tree. In this diagram, the tree mutated 4 times (as shown by each divergence of the tree at each node). The final 5 populations each have unique genotypes._ 

![Phylo Tree Diagram](/assets/BlogImages/PhylogeneticTreeCorrect.jpg)

### **Null Models**

A null model is a randomly generated model of an object or structure that is not constrained by its typical characteristics, and is instead based on the randomization of data and structure. A null model attempts to achieve the most unbiased model possible. 


### **Phylogenetic Diversity**

Phylogenetic diversity is a measure of biodiversity in a population or set of species. In this project, phylogenetic diversity is defined as the number of internal nodes in the tree plus the number of extant taxa, minus one. This metric assumes that all branches from parent to child have a length of one. Extant taxa are groups of organisms that are still present in the tree, and have not died out (become extinct). 

+++++ Reoccuring comma splice issue - check out the "Comma Before And That Joins Two Independent Clauses" section in this great how-to article: https://www.grammarly.com/blog/comma-before-and/ 

We decided that I would use phylogenetic diversity as our metric for comparison. I could have also used evolutionary distinctiveness, however, phylogenetic diversity is a highly applicable trait among trees, and it is easy to calculate, making it a desirable metric for comparison. 

+++++ Active voice & avoiding subject/verb seperation "We used phylogenetic diversity as the comparison metric. While evolutionary distinctiveness would also work, phylogenetic diversity is a more desirable metric for comparing trees, because it is a highly applicable trait among them, and it is easy to calculate.


--------------------------------------------------------------------------

### **The Null Model**

Coming up with a null model of a tree was not the most intuitive, but we decided that having the most randomly generated model was the best option. 
+++++nitpicky: word repetition of "most" that would be easy to fix by changing the first use to something like "coming up with a null model of a tree was difficult..."

We randomized how organisms were selected and how the tree branched as a result. 

The way that organisms were chosen for reproduction is shown here: 

```c++
int chooseOrg(vector<Organism> &currentGen, emp::Random &randNum){

    parentNum = randNum.GetInt(size(currentGen));  //chooses random spot 
    return parentNum;
}
```

Here the Empirical random number generator was utilized to ensure that results were actually random. First, a random number was generated based on the size of the parent generation. That spot in the array was then set as the parent of the next generation. 

In the null model, each time a new organism was created, it represented its own clade or taxon to ensure maximum diversity. 

### **Mutation and Pressure for Diversity** 

**Mutation**

The trees we used for comparison were trees with mutations and pressure for diversity. 
+++++Word repetition of "trees." -- "We used trees with mutations and pressure for diversity for compairison."

The mutation rate used for all of the trees was 0.05, which is a typical value for tree modeling in Empirical. In these models, each organism had a genotype as an attribute. 

Mutation was determined randomly. 
+++++ dangling modifier. Even though it's the way we speak, you technically shouldn't end a sentence with words ending in "-ly", so you'd want "was randomly determined." 

The population generated in the first round of the tree all had a genotype of integer 0. 
+++++ another Emily question - what is this "round of the tree" actually called? Generation? Level?

A random double between 0 and 1 was generated with each creation of an organism following this generation. If the value generated was less than 0.05, the genotype would mutate. If a mutation was required, a new random number would be generated between -3 and 3. That genotype would then be subtracted from the original genotype. 

For example, if the organism had a genotype of 2, was chosen to mutate, and the mutated genotype generated was -3, then the new genotype for that organism would be 2 - (-3), which is 5. 

Mutations are also heritable, meaning that an organism's child inherits its parent's mutated or not mutated genotype. Once an organism mutates, it creates a branch in the tree. 
+++++ edited for active voice and subject/verb seperation. I recommend checking out Google's page on active voice in tech writing: https://developers.google.com/tech-writing/one/active-voice?hl=zh-cn

The following code shows how the organism class handles mutations. In the model that just used mutations but did not account for any pressure to diversify, this code is used as the organism class, but has no fitness calculations and still uses random choice for the creation of child organisms.  

```c++
class Organism {
public:
    int genotype = 0;

    Organism() {
    }

    Organism(int _genotype) {
        genotype = _genotype;
    }

    int MutateGenotype(emp::Random &RandNum) {

        double randMutation = RandNum.GetDouble(0, 1);

        if (randMutation < mutRate) {
            int MutatedGenotype = genotype - RandNum.GetInt(-3, 3);
            genotype = MutatedGenotype;
            cout << "mutated genotype = " << genotype << endl;
        } else {
            cout << "not mutated genotype = " << genotype << endl;
        }

        return genotype;
    }
};
```

**Pressure for Diversity**

A goal of this project was to show that, when we add a constraint that incentivizes the tree to branch more frequently, the overall diversity increases. This was accomplished by adding a pressure for the tree to diversify by favoring rarer genotypes. 

In the model that uses pressure for diversity and mutations, genotypes that are rarer are favored for reproduction over more common genotypes. When rarer genotypes are chosen, diversity increases throughout the tree. We refer to this as fitness and calculate it in the following code: 

```c++
void calcFitness(vector<Organism> &currentGen, vector<double> &fitnessVect, emp::Random &randNum) {
    fitnessVect.resize(0);

    vector<int> fitnessCalc;

    fitnessCalc.reserve(currentGen.size());
    for (int i = 0; i < currentGen.size(); i++) {
        fitnessCalc.push_back(currentGen[i].genotype);
    }

    map<int, int> CountMap;

    for (int j = 0; j < fitnessCalc.size(); j++) {
        if (emp::Has(CountMap, fitnessCalc[j])) {
            CountMap[fitnessCalc[j]]++;
        } else {
            CountMap[fitnessCalc[j]] = 1;

        }
    }

    for(int k = 0; k < fitnessCalc.size(); k++){
        fitnessVect.push_back(1.0/CountMap[fitnessCalc[k]]);
    }
}
```

**_THIS CODE NEEDS COMMENTING AND EXPLANATION_**

After fitness was calculated, the organisms with rarer fitness were chosen for reproduction. For this, I used the following ChooseOrgDiversity function. 
+++++ you alternate back and forth between current & past tense when describing what the code does. You want current, for active voice, like "After fitness is calculated..."

```c++
int chooseOrgDiversity(vector<double> &fitnessVect, emp::Random &randNum){
    emp::IndexMap fitness_index(fitnessVect.size());

    for (size_t id = 0; id < fitnessVect.size(); id++){
        fitness_index.Adjust(id, fitnessVect[id]);
    }

    const double fit_pos = randNum.GetDouble(fitness_index.GetWeight());
    size_t parent_id = fitness_index.Index(fit_pos);

    //cout << "FITNESS VECTOR VALUES: " << endl;

    for(int pos = 0; pos < fitnessVect.size(); pos++){
        //cout << fitnessVect[pos] << " " << endl;
    }

    parentNum = parent_id;

    cout << "PARENT NUM AFTER CHOOSEORGDIVERSITY: " << parentNum << endl;
    cout << "fitness val at parent_id: " << fitnessVect[parent_id] << endl;

    return parentNum;
}
```

In all three of these models, phylogenetic diversity increases with tree depth (number of generations). This is very clearly illustrated in all 3 of my models and is shown in the graph below. 
+++++Again nitpicky, but words like "very" are typically avoided in tech and science writing, because it doesn't add anything new, so it's better to leave it out to be concise. "This is clearly illustrated in all 3 of my models..." stands well on its own :) 

![Average Phylogenetic Diversity Over Time for All Three Models](/assets/BlogImages/DiversityOverTime.jpg)

### **Systematics** 

Systematics.h is file manager in Empirical. It is used to track genotypes, species, clades, or lineages of organisms. Systematics.h allows a user to create phylogenetic trees with various levels of abstraction -- using genotypes, phenotypes, etc. to keep track of lineage. 
+++++Should "Systematics.h" be formated the way you format method names since it's the class name? (I don't actually know...)

This project focused on two topics -- creating models to establish the possible range of phylogenetic diversity, then testing those models, and lastly, incorporating these percentiles into systematics so that a user could find out how their own trees compare. 
+++++Clarity: reading this, it sounds like 3 topics. Maybe move testing so it isn't on its own, like "creating & testing models to establish the..."

Within the systematics manager, I added two functions to use when calculating phylogenetic diversity and finding the percentile associated with that diversity. 

The first function, ```FindPhyloData()```, is used to compare results with the null model. It calculates the phylogenetic diversity wherever the function is called and returns the percentile corresponding to that value based on the data from the null model, which is stored in tree_percentiles.csv. 
+++++clarity: maybe "returns the percentile corresponding to that value based on the null model data stored in tree_percentiles.csv." 

**_in the final draft the included code will be simplified and commented more_**

```c++
  int FindPhyloData(){
    int percentile; 

    emp::File tree_percentiles("tree_percentiles.csv"); //loading file

    emp::vector< emp::vector<double> > percentile_data = tree_percentiles.ToData<double>(','); //turns data into an array

     int PhyloDiversity = GetPhylogeneticDiversity(); 

    for (int i = 0; i < percentile_data.size() - 1; i++){ 

        if( (PhyloDiversity >= percentile_data[i][1]) && (PhyloDiversity < percentile_data[i + 1][1])){ 
           std::cout << "Phylogenetic Diversity (recorded in systematics): " << PhyloDiversity << std::endl; 
           std::cout << "phylo diversity is in between: " << percentile_data[i][1] << " and " << percentile_data[i + 1][1] << std::endl; 
           std::cout << PhyloDiversity << " is in percentile: " << percentile_data[i][0] << std::endl;       

           percentile = percentile_data[i][0];

           std::cout << percentile << std::endl; 
           }
      }
      return percentile; 
    }
```

The following function is used for trees that contain pressure for diversity or mutations. It can also be used for multiple generations (10 through 100 gens). When called, it takes an argument of the number of generations. 
++++This is perfect! This is the tense you want when talking about your code.

This corresponds to a line in OrgGenotypePercentiles.csv, each containing percentiles for different numbers of generations. This function only allows users to use multiples of 10 for the generation numbers. For example, 10, 20, 30, ... 100 generations. 

_Right now, this function is used for testing our findings and prints the data to PercentileDataFullNoPressure.csv. This will probably be changed in the final product._

```c++
    void FindPhyloMultipleGens(int GenValueRaw){ 
      int GenValue = ((GenValueRaw / 10) - 1); 
      int percentile; 
      bool percentFound = false; 

        emp::File generation_percentiles("OrgGenotypePercentiles.csv");
        emp::vector< emp::vector<double> >percentile_data2 = generation_percentiles.ToData<double>(',');

      int PhyloDiversity = GetPhylogeneticDiversity(); 
      int lastval = size(percentile_data2[GenValue]) - 1; 
      std::cout << "Last element of array is: " << percentile_data2[GenValue][lastval] << std::endl;

      std::fstream fs; 
      fs.open("PercentileDataFullNoPressure.csv", std::fstream::in | std::fstream::out | std::fstream::app);

        //for(int i = 0; i < percentile_data2.size() - 1; i++){ 
          for(int j = 0; j <= percentile_data2[GenValue].size() - 2; j++){
          
          if((percentile_data2[GenValue][j] <= PhyloDiversity) && (percentile_data2[GenValue][j + 1] > PhyloDiversity)){
            std::cout << "phylo diversity is in between: " << percentile_data2[GenValue][j] << "and " << percentile_data2[GenValue][j+1] << std::endl;
            std::cout << "I is equal to: " << GenValue << std::endl; 
            std::cout << "J is equal to: " << j << std::endl;

            std::cout << "The Phylogentic diversity value " << PhyloDiversity << " is in the " << j << " percentile, in the " << ((GenValue + 1)* 10) << " generation" << std::endl;  

            fs << ((GenValue + 1)* 10) << "," << j << std::endl; 

            percentFound = true; 
          }
          
          if(PhyloDiversity >= percentile_data2[GenValue][lastval]){ 
              fs << ((GenValue + 1) * 10) << "," << 100 << std::endl; 
              fs.close(); 
            }

        if(percentFound == true){ 
            break; 
          }
          }
          if(percentFound == false){ 
            std::cout << "PHYLO DIVERSITY IS IN 100TH PERCENTILE" << std::endl; 
           }
        }
```

------------------------- 
## **Method**

I ran each of the three models 1000 times for every 10 generations and collected the phylogenetic diversity values at the end of each run.

For example, I would set the number of generations in the null model to 10. Next, I would run it 1000 times, recording the final diversity at the end of each run. I would then do 20 generations, and so forth, all the way through 100 generations. 

After the initial data collection, I ran the data through a Python script which created a percentile range. It did this by first sorting all of the data in ascending order. It would then take every 10th value in the dataset, to output 100 final diversity values, each corresponding to a percentile value from 0 to 100. I repeated the same process for each model.

To incorporate this data into the systematics manager, I imported the percentile CSV files into the two functions described in the systematics section above. Wherever these functions are called in future code, they will calculate the phylogenetic diversity of the tree and return the percentile value for how the tree compares to the models. 

After I setup this framework, I decided to test its reliability. I ran my models once again and had the systematics manager classify each tree's final phylogenetic diversity after each set of generations. I used the file containing percentiles for the tree that used mutations but had no pressure for diversity. 

When I used this process on the tree with mutations but with no pressure for diversity, I would expect to see values in the 50th percentile range. However, if repeated for the tree with a pressure for diversity, I would expect values near the 100th percentile. 

## **Results**

**The following table contains the percentile values for a tree with _no_ pressure for diversity**

|          |    |    |    |    |    |    |    |    |    |    | Average | Standard Deviation |
|----------|----|----|----|----|----|----|----|----|----|----|---------|--------------------|
| 10 gens  | 61 | 37 | 61 | 97 | 61 | 61 | 61 | 61 | 37 | 37 |    57.4 |        17.93320942 |
| 20 gens  | 88 | 95 | 99 | 19 | 95 | 88 | 40 | 62 | 95 | 88 |    76.9 |        27.44064301 |
| 30 gens  | 16 | 95 | 97 | 36 | 98 | 95 | 87 | 36 | 75 | 57 |    69.2 |        30.67318771 |
| 40 gens  | 11 | 90 | 68 | 11 | 94 | 81 | 90 | 11 | 50 | 11 |    51.7 |        37.23215456 |
| 50 gens  |  6 | 74 | 58 | 58 | 97 | 39 | 93 | 39 | 74 | 20 |    55.8 |        29.96219841 |
| 60 gens  | 16 | 90 | 31 | 68 | 50 | 31 | 98 | 68 | 50 | 68 |      57 |        26.38181192 |
| 70 gens  | 26 | 93 | 26 | 26 | 43 | 61 | 93 | 13 | 61 | 13 |    45.5 |        30.28109054 |
| 80 gens  | 69 | 81 | 20 | 36 | 69 | 69 | 98 |  9 | 69 | 20 |      54 |        30.23243292 |
| 90 gens  | 17 | 61 | 17 | 31 | 75 | 61 | 96 | 47 | 61 | 47 |    51.3 |        25.04240847 |
| 100 gens | 12 | 70 |  1 | 39 | 39 | 56 | 98 | 25 | 70 | 39 |    44.9 |        29.27437256 |


**The table below contains the percentile values for a tree _with_ pressure for diversity.**
|          |     |    |     |     |     |     |     |     |     |     | Average | Standard Deviation |
|----------|-----|----|-----|-----|-----|-----|-----|-----|-----|-----|---------|--------------------|
| 10 gens  |  37 | 80 |  80 |  61 |  99 |  61 | 98  | 91  | 61  |  91 |    75.9 |         20.2509259 |
| 20 gens  |  62 | 77 |  98 |  95 |  99 |  88 | 99  | 77  | 77  |  98 |      87 |        12.99572579 |
| 30 gens  |  57 | 95 |  98 |  98 |  97 |  97 | 99  | 95  | 95  |  97 |    92.8 |        12.65613597 |
| 40 gens  | 100 | 94 |  99 |  90 |  97 |  97 | 100 | 97  | 97  | 100 |    97.1 |        3.142893218 |
| 50 gens  |  97 | 74 | 100 |  93 | 100 |  99 | 93  | 97  | 99  | 100 |    95.2 |        7.913420387 |
| 60 gens  |  99 | 95 |  95 | 100 |  99 |  98 | 98  | 95  | 99  | 100 |    97.8 |        2.043961296 |
| 70 gens  |  99 | 99 |  93 | 100 |  99 |  99 | 99  | 99  | 99  | 100 |    98.6 |        2.011080417 |
| 80 gens  |  99 | 95 |  99 |  98 |  99 | 100 | 81  | 100 | 100 |  99 |      97 |        5.811865258 |
| 90 gens  |  99 | 96 |  99 |  99 |  98 |  98 | 86  | 99  | 99  |  99 |    97.2 |        4.049691346 |
| 100 gens |  99 | 99 |  99 |  99 |  98 |  99 | 94  | 99  | 99  |  99 |    98.4 |        1.577621275 |

![Percentile Graph](/assets/BlogImages/PercentileGraph.jpg)

These tables and graph show that the tree with no pressure for diversity outputs values that average at 56.37, around what we would expect for this model. The trees with pressure for diversity, regardless of tree depth, average 93.7, which is very close to the expected outcome. These results illustrate that the percentile data collected can be used to classify future trees between 10 and 100 generations. 

## **Conclusion**

Based on the results shown above, the current function in the systematics manager, FindPhyloMultipleGens(), is able to accurately return the percentile value of a given tree with a relatively high accuracy rate.
++++format function name

Considering the size of this project, the standard deviation observed in our findings does not raise much concern for validity. The results seen, being that trees with no pressure for diversity result in an average classification of **56.37** and trees with pressure to diversify have an average classification of **93.7**, we can conclude that this function provides an accurate classification of phylogenetic trees. 
+++++This last sentence is awkwardly phrased. Maybe a good one to rework in our last meeting?

If I were to continue this project, I would write several more tests to improve the validity of these findings. I would also find percentile classifications for a much larger sample size for a more accurate data set. 

This workshop was an incredible learning experience for me, and I am very excited that I was able to generate meaningful data and tools for the Empirical library. I came into this workshop looking to improve my scientific computing and programming skills, and I can safely say that this workshop has provided me with a unique and invaluable way to do so. I am so grateful to my mentors, Emily and Kate, for their patience and guidance throughout this process. Thank you to the entire WAVES team for their flexibility and commitment to this program and to their participants, especially in such an unprecedented situation. Thank you WAVES for such an incredible opportunity! 

###### This work is supported through Active LENS: Learning Evolution and the Nature of Science using Evolution in Action (NSF IUSE #1432563). Any opinions, findings, and conclusions or recommendations expressed in this material are those of the author(s) and do not necessarily reflect the views of the National Science Foundation.


