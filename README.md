# Essential Term Selector 

## About 

This project contains a supervised learning classifier for identifying *essential* terms 
in a given a question (terms which are defintely needed for answering a question). 
Here is an example here are types of the predictions you'd get from our system for the input question 
"In New York State, the longest period of daylight occurs during which month? (A) December (B) June (C) March (D) September": 

```
 Map(In -> 0.0, daylight -> 0.7142422913303911, , -> 0.0, occurs -> 0.4903305931093945, York -> 0.5033343254566864, ? -> 0.0, longest -> 0.6679789993588322, which -> 0.0, New -> 0.4733048916171777, State -> 0.4170088051802153, during -> 0.0, of -> 0.0, period -> 0.6333389494193361, month -> 0.6438509592140552, the -> 0.0)
```

The main input for supervised training is collected via mechanical turk 
and is included in the project.  
 
To see more comprehensive analysis on our classifier and its usages 
in other QA systems, have a brief look at [1]. 

## Dataset 
Checkout the [dataset folder](data). 

## Compiling the code 
To compile the code: 
```
 > sbt compile
```

## Using it in your system 
First you have to include it as a dependency in your project. Here is how it's done for an sbt project: 

```sbt
resolvers += "CogcompSoftware" at "http://cogcomp.cs.illinois.edu/m2repo/"
libraryDependencies += "org.allenai.ari" %% "essential-terms" % "1."
```
Next we show how to use it in your program: 

### Using it as a service via `Injector` library
If you want to use Google's injector library, many of the minor details will automatically be taken care of. 
You just need to inject the service defined in `EssentialTermsService`. 

```scala 
val essentialTermService = ... // injected essential-terms service
 
// first decompose question 
val q = "In New York State, the longest period of daylight occurs during which month? (A) " +
       "December (B) June (C) March (D) September"
val aristoQuestion = Utils.decomposeQuestion(q)

// and make prediction
// only the essential terms
val essentialTerms = essentialTermService.getEssentialTerms(aristoQuestion)
// the essentiality scores 
val essentialTermScores = essentialTermService.getEssentialTermScores(aristoQuestion)
```
  
### Direct usage 
    
We have to initialize the classifiers and make predictions on a given question
   
```scala 
// loading the models
val salienceBaselines = SalienceLearner.makeNewLearners()
val (baselineDataModel, baselineClassifiers) = BaselineLearners.makeNewLearners(LoadFromDatastore, "dev")
val (expandedDataModel, expandedLearner) = ExpandedLearner.makeNewLearner(LoadFromDatastore, "SVM", baselineClassifiers, baselineDataModel, salienceBaselines)
 
// read the raw question and decompose it 
val q = "In New York State, the longest period of daylight occurs during which month? (A) " +
       "December (B) June (C) March (D) September"
val aristoQuestion = Utils.decomposeQuestion(q)

// make predictions
// essential terms 
val essentialTerms = expandedLearner.getEssentialTerms(aristoQuestion, threshold = Constants.EXPANDED_LEARNER_THRESHOLD)
// essentiality scores 
val essentialTermsScores = expandedLearner.getEssentialTermScores(aristoQuestion, threshold = Constants.EXPANDED_LEARNER_THRESHOLD)
```
  
## Further Reading 
Checkout the following paper for more details and analysis 

[1] Daniel Khashabi, Tushar Khot, Ashish Sabharwal and Dan Roth, "Learning What is Essential in Questions", Conference on Natural Language Learning (CoNLL), 2017.
