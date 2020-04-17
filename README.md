
<img src="1200px-University_of_Tehran_logo.svg.png" width="100" style="float:left;"/>

<img src="fanni.png" width="120" style="float:right;position: relative;top: -25px;"/>


<h1 style="float:center;" align="center">Computer Assignment 2</h1>
<h4 style="float:center;" align="center"><b> Navid Akbari ( 810895023 ) </b></h4>

<br>

The goal of this computer assignment is to get more familiar with the genetic algorithm for searching. This algorithm is used when we have a state explosion problem, and our space is very large. In this assignment, we try to decipher substitution cryptography. If we want to try all possible states, we should check $26!$ states, which is impossible. But with the genetic algorithm, we will solve this problem in a timely manner way.


```python
import string, time, re, random
import numpy as np
from operator import itemgetter
```

## Definition of Chromosome


I define a chromosome as a map from the actual alphabet to the replaced alphabet. I choose this representation because firstly, its more understandable, and secondly, map data structure is more efficient for finding and replacing elements. One example of this:

`{'a': 'o', 'b': 'r', 'c': 's', 'd': 'f', 'e': 'w', 'f': 'm', 'g': 'b', 'h': 't', 'i': 'i', 'j': 'z', 'k': 'g', 'l': 'h', 'm': 'k', 'n': 'n', 'o': 'v', 'p': 'e', 'q': 'l', 'r': 'p', 's': 'd', 't': 'j', 'u': 'c', 'v': 'u', 'w': 'y', 'x': 'q', 'y': 'a', 'z': 'x'}`


## Making Primitive Population

At the beginning of the algorithm, I make a primitive population with a size of `POPULATION_SIZE`, which is a constant in top of the code by using `makeRandomChromosome` function. In this function, I make a random chromosome by using `random` function.

## Cleaning Data

For cleaning data, I used `cleanData` function. In this function, I first replace all things except alphabet with space and then split data with space, and finally, I convert a list of words to map to delete the duplicated words. The problem with stop words is their rate of occurrence, so I remove the duplicated words. So now we have a dictionary which we can used it to find the fitness of each Chromosome.

## Fitness Function

For finding the fitness value of each chromosome, I used `fitness` function. In this function, I calculate the deciphered text and then calculate how many words in the deciphered text is in the dictionary. The number of words is the fitness value for each chromosome. Then, I sort them in reverse mode because I want to choose the best of them for the next generation. I will explain it more.

## Selection

I use rank base selection for this project, because of the `FPS` problem in making chromosome bios. Also, I use **ELITISM** selection method. This means I copy 20% of the best chromosomes to the next generation, the remain, 80%, used for crossover and mutation.

#### What if we increase the population size in each round?
This population increase doesn't help our algorithm because the number of generations is very big, and it makes the population very large and makes the processing time a lot.

## Crossover

For the implementation of crossover, I use the order 1 method. In the `crossover` method, I choose two different points and then copy the values between these points to the offspring and then complete the chromosome. The crossover happens in 80% of the population. I choose the parents based on their rank.

## Mutation

For mutation, the result of the crossover is used for this method. In `mutation` function, I choose a random number between 0 to 5, and base on this number, I swap the mapping of the chromosome.

#### What if we just use crossover?
If we use just crossover in the genetic algorithm, its very potential to stuck in the local extremum for a long time. So, the mutation is an important part of the genetic algorithm.

#### Mutation is more effective or crossover? Which of them lead to more accuracy?
Crossover leads to making a new generation. It makes completely new solutions and leads to significant improvement in the fitness scores. But, as I said, using just crossover may lead to stuck in some situations. So, mutation helps to change the current answer a little to make a possibly better generation.

#### Why chromosomes get stuck in some situations?
Because they stuck in the local extremum.

#### How do we solve this problem?
I use mutation multiple times. I use a random number between 0 to 5 for swap the genes.



```python
POPULATION_SIZE = 100
ELITISM_RATE = 0.2
CROSSOVER_RATE = 0.8

class Decoder:
    
    def __init__(self, encodedText):
        self.encodedText = encodedText
        self.encodedTextWords = self.cleanData(self.encodedText.lower())
        self.dictionary = {}
        self.createDictionary()
         
            
    def makeRandomChromosome(self):
        chars = set(string.ascii_lowercase)
        map = {}
        for c in string.ascii_lowercase:
            char = random.choice(list(chars))
            chars.remove(char)
            map[c] = char
        return map
    
    
    def decipher(self, key):
        decipheredText = ''
        for char in self.encodedText:
            if not char.islower():
                decipheredText += key.get(char.lower(), char).upper()
            else:    
                decipheredText += key.get(char, char)
        return decipheredText
    
    
    def decipherWords(self, key):
        decipheredText = list()
        for word in self.encodedTextWords:
            temp = ''
            for char in word:
                temp += key.get(char, char)
            decipheredText.append(temp)
        return decipheredText
    
    
    def fitness(self, chromosome):
        decipheredWords = self.decipherWords(chromosome)
        
        counter = 0
        for word in decipheredWords:
            if word in self.dictionary:
                counter += 1
                
        return counter
    
    
    def cleanData(self, dataSet):
        dataSet = re.sub(r'[^A-Za-z]', ' ', dataSet)
        dataSetWords = dataSet.split()
        dataSetWords = list(dict.fromkeys(dataSetWords))
        return dataSetWords
    
    
    def createDictionary(self):
        dataSet = open("global_text.txt").read().lower()
        dataSetWords = self.cleanData(dataSet)
        
        self.dictionary = set(dataSetWords)
            
    def makeChromosome(self, Chromosome):
        map = {}
        for c in string.ascii_lowercase:
            char = Chromosome[0]
            Chromosome.remove(char)
            map[c] = char
        return map
            
        
    def makeChild(self, first, second):
        size = len(first)
        point1 = random.randint(1, size - 1)
        point2 = random.randint(1, size - 1)
        if point2 >= point1:
            point2 += 1
        else:
            point1, point2 = point2, point1
        
        temp = first[point1:point2]
        second = [x for x in second if x not in temp]
        newChromosome = second[:point1] + temp + second[point1:]
        
        return self.makeChromosome(newChromosome)    
           
        
    def crossover(self, father, mother):
        fatherValues = list(father.values())
        motherValues = list(mother.values())
        
        firstChild = self.makeChild(fatherValues, motherValues)

        return firstChild
            
        
    def calFitnesses(self, population):
        populationScores = [[self.fitness(population[i]), i] for i in range(len(population))]
        populationScores = sorted(populationScores, key=itemgetter(0), reverse = True)
        return populationScores
    
    
    def mutation(self, chromosome):
        size = len(list(chromosome.values()))
        for i in range(random.randint(0, 5)):
            point1 = random.randint(0, size - 1)
            point2 = random.randint(0, size - 1)

            values = list(chromosome.values())
            keys = list(chromosome.keys())
            chromosome[keys[point1]], chromosome[keys[point2]] = values[point2], values[point1]

        return chromosome
    
    
    def geneticAlgorithm(self):
        population = [self.makeRandomChromosome() for i in range(POPULATION_SIZE)]
        
        sumOfRanks = (len(population) * len(population) + 1)/2
        ranksProbabilities = [(i+1)/sumOfRanks for i in range(len(population))]
        ranksProbabilities = sorted(ranksProbabilities, reverse = True)
       
        while True:
            populationScores = self.calFitnesses(population)
            
            if populationScores[0][0] >= len(self.encodedTextWords):
                chromosome = population[populationScores[0][1]]
                print("The answer Chromosome is: ")
                print(chromosome , "\n")
                
                return self.decipher(chromosome)

          
            newPopulation = []
            
            newPopulation.extend([population[populationScores[i][1]] for i in range(int(POPULATION_SIZE*ELITISM_RATE))])
            
            size = int(CROSSOVER_RATE*POPULATION_SIZE) 
            
            for i in range(size):
                parent = np.random.choice(population,2 , ranksProbabilities) 
                offspring = self.crossover(parent[0] , parent[1]) 
                newPopulation.append(self.mutation(offspring))
            
            population = newPopulation
            
            
    def decode(self):
        return self.geneticAlgorithm()
        
```


```python
encodedText = open("encoded_text.txt").read()

d = Decoder(encodedText)

start = time.time()
decodedText = d.decode()
end = time.time()

print("Time: %s seconds" % (end - start) , "\n")
print("The decoded text is: \n")
print(decodedText)

```

    The answer Chromosome is: 
    {'a': 'o', 'b': 'r', 'c': 's', 'd': 'f', 'e': 'w', 'f': 'm', 'g': 'b', 'h': 't', 'i': 'i', 'j': 'z', 'k': 'g', 'l': 'h', 'm': 'k', 'n': 'n', 'o': 'v', 'p': 'e', 'q': 'l', 'r': 'p', 's': 'd', 't': 'j', 'u': 'c', 'v': 'u', 'w': 'y', 'x': 'q', 'y': 'a', 'z': 'x'} 
    
    Time: 71.19295716285706 seconds 
    
    The decoded text is: 
    
    This response originally fell into a bit bucket.  I'm reposting it
    just so Bill doesn't think I'm ignoring him.
    
    In article <C4w5pv.JxD@darkside.osrhe.uoknor.edu> bil@okcforum.osrhe.edu (Bill Conner) writes:
    >Jim Perry (perry@dsinc.com) wrote:
    >
    >[Some stuff about Biblical morality, though Bill's quote of me had little
    > to do with what he goes on to say]
    
    Bill,
    
    I'm sorry to have been busy lately and only just be getting around to
    this.
    
    Apparently you have some fundamental confusions about atheism; I think
    many of these are well addressed in the famous FAQ.  Your generalisms
    are then misplaced -- atheism needn't imply materialism, or the lack
    of an absolute moral system.  However, I do tend to materialism and
    don't believe in absolute morality, so I'll answer your questions.
    
    >How then can an atheist judge value? 
    
    An atheist judges value in the same way that a theist does: according
    to a personal understanding of morality.  That I don't believe in an
    absolute one doesn't mean that I don't have one.  I'm just explicit,
    as in the line of postings you followed up, that when I express
    judgment on a moral issue I am basing my judgment on my own code
    rather than claiming that it is in some absolute sense good or bad.
    My moral code is not particular different from that of others around
    me, be they Christians, Muslims, or atheists.  So when I say that I
    object to genocide, I'm not expressing anything particularly out of
    line with what my society holds.
    
    If your were to ask why I think morality exists and has the form it
    does, my answer would be mechanistic to your taste -- that a moral
    code is a prerequisite for a functioning society, and that humanity
    probably evolved morality as we know it as part of the evolution of
    our ability to exist in large societies, thereby achieving
    considerable survival advantages.  You'd probably say that God just
    made the rules.  Neither of us can convince the other, but we share a
    common understanding about many moral issues.  You think you get it
    from your religion, I think I get it (and you get it) from early
    childhood teaching.
    
    >That you don't like what God told people to do says nothing about God
    >or God's commands, it says only that there was an electrical event in your
    >nervous system that created an emotional state that your mind coupled
    >with a pre-existing thought-set to form that reaction. 
    
    I think you've been reading the wrong sort of comic books, but in
    prying through the gobbledygook I basically agree with what you're
    saying.  I do believe that my mental reactions to stimuli such as "God
    commanded the genocide of the Canaanites" is mechanistic, but of
    course I think that's true of you as well.  My reaction has little to
    do with whether God exists or even with whether I think he does, but
    if a god existed who commanded genocide, I could not consider him
    good, which is supposedly an attribute of God.
    
    >All of this being so, you have excluded
    >yourself from any discussion of values, right, wrong, goood, evil,
    >etc. and cannot participate. Your opinion about the Bible can have no
    >weight whatsoever.
    
    Hmm.  Yes, I think some heavy FAQ-reading would do you some good.  I
    have as much place discussing values etc. as any other person.  In
    fact, I can actually accomplish something in such a discussion, by
    framing the questions in terms of reason: for instance, it is clear
    that in an environment where neighboring tribes periodically attempt
    to wipe each other out based on imagined divine commands, then the
    quality of life will be generally poor, so a system that fosters
    coexistence is superior, if quality of life is an agreed goal.  An
    absolutist, on the other hand, can only thump those portions of a
    Bible they happen to agree with, and say "this is good", even if the
    act in question is unequivocally bad by the standards of everyone in
    the discussion.  The attempt to define someone or a group of people as
    "excluded from discussion", such that they "cannot participate", and
    their opinions given "no weight whatsoever" is the lowest form or
    reasoning (ad hominem/poisoning the well), and presumably the resort
    of someone who can't rationally defend their own ideas of right,
    wrong, and the Bible.
    -- 
    Jim Perry   perry@dsinc.com   Decision Support, Inc., Matthews NC
    These are my opinions.  For a nominal fee, they can be yours.
    
    -- 
    Jim Perry   perry@dsinc.com   Decision Support, Inc., Matthews NC
    These are my opinions.  For a nominal fee, they can be yours.
    

