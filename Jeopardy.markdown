# Jeopardy: Probability and Statistics

**Analyzing text and exploring strategies to gain a competitive edge.**  

# 1. About Jeopardy!

Jeopardy! is an American television game show created by Merv Griffin. The show features a quiz competition in which contestants are presented with general knowledge clues in the form of answers, and must phrase their responses in the form of questions. Three contestants, including the previous show's champion, compete in a quiz game comprising three rounds: Jeopardy!, Double Jeopardy!, and Final Jeopardy!. The material for the clues covers a wide variety of topics, including history and current events, the sciences, the arts, popular culture, literature, and languages. 

The first episode aired on: September 10, 1984

# 2. Exploring the Data 
To increase our chances of winning, we will analyze content from previously aired episodes. To do this we will scrape our data from 
[J-Archive](http://www.j-archive.com/), a fan-created archive of Jeopardy! games. Our dataset includes the following columns:

| Columns  | Definition |
| :------------ | :------------- |
| **Show Number**  | Jeopardy episode from which the question appeared. |
| **Air Date**     | The date the episode aired. |
| **Round**        | The Jeopardy round the question was asked in. |
| **Category**     | The category of the question. |
| **Value**        | The value gained or lost depending on answering correctly. |
| **Question**     | The question being asked. |
| **Answer**       | The correct answer to the question. |


```python
import csv
import numpy as np
import pandas as pd

# Read the dataset into a Dataframe called jeopardy
jeopardy = pd.read_csv("jeopardy.csv")

# Print the number of rows and columns
print(jeopardy.shape)

# Print out the first 5 rows of jeopardy
jeopardy.head()
```

    (19999, 7)





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Show Number</th>
      <th>Air Date</th>
      <th>Round</th>
      <th>Category</th>
      <th>Value</th>
      <th>Question</th>
      <th>Answer</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4680</td>
      <td>2004-12-31</td>
      <td>Jeopardy!</td>
      <td>HISTORY</td>
      <td>$200</td>
      <td>For the last 8 years of his life, Galileo was ...</td>
      <td>Copernicus</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4680</td>
      <td>2004-12-31</td>
      <td>Jeopardy!</td>
      <td>ESPN's TOP 10 ALL-TIME ATHLETES</td>
      <td>$200</td>
      <td>No. 2: 1912 Olympian; football star at Carlisl...</td>
      <td>Jim Thorpe</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4680</td>
      <td>2004-12-31</td>
      <td>Jeopardy!</td>
      <td>EVERYBODY TALKS ABOUT IT...</td>
      <td>$200</td>
      <td>The city of Yuma in this state has a record av...</td>
      <td>Arizona</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4680</td>
      <td>2004-12-31</td>
      <td>Jeopardy!</td>
      <td>THE COMPANY LINE</td>
      <td>$200</td>
      <td>In 1963, live on "The Art Linkletter Show", th...</td>
      <td>McDonald's</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4680</td>
      <td>2004-12-31</td>
      <td>Jeopardy!</td>
      <td>EPITAPHS &amp; TRIBUTES</td>
      <td>$200</td>
      <td>Signer of the Dec. of Indep., framer of the Co...</td>
      <td>John Adams</td>
    </tr>
  </tbody>
</table>
</div>



As you can see each row in the dataset represents a single question on a single episode of Jeopardy. Further observing the format of the data, we can see that our column names have unwanted spaces around them and need to be cleaned up.


```python
# List out the columns of jeopardy using jeopardy.columns
list(jeopardy.columns)
```




    ['Show Number',
     ' Air Date',
     ' Round',
     ' Category',
     ' Value',
     ' Question',
     ' Answer']



We will remove the spaces in each item in jeopardy.columns, and then assign the result back to jeopardy.columns to fix the column names. 


```python
jeopardy.columns = ['Show Number', 'Air Date', 'Round', 'Category', 'Value', 'Question', 'Answer']
```

# 3. Normalizing the Data

Before we can start doing any analysis on the Jeopardy questions, we need to normalize all of the text columns (the `Question` and `Answer` columns). 

What we want to do is lowercase all of the words and remove punctuation so `Don't` and `don't` aren't considered to be different words when we compare them.  

Our function to normalize the text will:  
- Take in a string
- Convert the string to lowercase
- Remove all punctuation in the string
- Return the string


```python
import re

def normalize_text(text):
    text = text.lower()
    text = re.sub("[^A-Za-z0-9\s]", "", text)
    return text
```

We can now apply our function to the `Question` and `Answer` columns and assign the resulsts to newly created columns `clean_questions` and `clean_answers` respectfully.


```python
jeopardy["clean_question"] = jeopardy["Question"].apply(normalize_text)
```


```python
jeopardy["clean_answer"] = jeopardy["Answer"].apply(normalize_text)
```

Our text columns are now normalized (`Questions` and `Answers`), but we still have more columns to take care of. The `Value` column needs to be numeric to allow us to appropriately work with our data. We will create a function that will:
- Take in a string
- Remove any punctuation in the string
- Convert the string to an integer
- If the conversion has an error, assign 0 instead
- Return the integer


```python
def normalize_values(text):
    text = re.sub("[^A-Za-z0-9\s]", "", text)
    try:
        text = int(text)
    except Exception:
        text = 0
    return text
```

We can now apply our function to the `Value` column and asign the results to a newly created column `clean_value`.


```python
jeopardy["clean_value"] = jeopardy["Value"].apply(normalize_values)
```

Finally, we can convert the `Air Date` column to the correct data type, datetime, using the `pandas.to_datetime` function.


```python
jeopardy["Air Date"] = pd.to_datetime(jeopardy["Air Date"])
```

Here is a look at each columns data type:


```python
jeopardy.dtypes
```




    Show Number                int64
    Air Date          datetime64[ns]
    Round                     object
    Category                  object
    Value                     object
    Question                  object
    Answer                    object
    clean_question            object
    clean_answer              object
    clean_value                int64
    dtype: object



# 4. Forming a Strategy

In order to figure out whether to study past questions, study general knowledge, or not study it all, we will start by trying to answer the following questions:  

**1. How often the answer is deducible from the question?**  
- We will be looking to find out how many times words in the answer also occur in the question  
  
**2. How often new questions are repeats of older questions?** 
- We will do this by seeing how often complex words (`> 6 characters`) reoccur 

## 4.1 Cross Referencing Answers and Questions
Our function to solve question 1:


```python
def count_matches(row):
    # Split the clean_answer column on the space character, and assign to the variable split_answer
    split_answer = row["clean_answer"].split(" ")
    # Split the clean_question column on the space character, and assign to the variable split_question
    split_question = row["clean_question"].split(" ")
    
    # If "the" is in split_answer, remove it using the remove method on lists
    if "the" in split_answer:
        split_answer.remove("the")
        
    # If the length of split_answer is 0, return 0. This prevents a division by zero error later 
    if len(split_answer) == 0:
        return 0
    match_count = 0
    # Loop through each item in split_answer, and see if it occurs in split_question. If it does, add 1 to match_count 
    for item in split_answer:
        if item in split_question:
            match_count += 1
            
    # Divide match_count by the length of split_answer, and return the result 
    return match_count / len(split_answer)

'''
Count how many times terms in clean_answer occur in clean_question.  

Use the Pandas apply method on Dataframes to apply the function to each row in jeopardy.  

Pass the axis=1 argument to apply the function across each row.  

Assign the result to the answer_in_question column. 
'''
jeopardy["answer_in_question"] = jeopardy.apply(count_matches, axis=1)
```


```python
# Find the mean of the answer_in_question column using the mean method on Series  
jeopardy["answer_in_question"].mean()
```




    0.060493257069335914



The answer only appears in the question about `6%` of the time and it doesn't look like we'll get much use from this approach. Let's take a look at recycled questions.

## 4.2 Question Overlap

For question 2, we need to investigate how often new questions are repeats of older ones by creating a function that:  

- Sorts our data in ascending order by the `Air Date` column
- Create a set called `terms_used` that will be used to cross reference the occurance of a word
- Iterate through each row of jeopardy
- Remove any word shorter than 5 characters to filter out words like `the` and `than`, that are commonly used, but don't tell us much about the question
- Increment a counter for word occurance
- Add each word to `terms_used`


```python
# Create an empty list called question_overlap
question_overlap = []
# Create an empty set called terms_used
terms_used = set()

# Use the iterrows DataFrame method to loop through each row of jeopardy
for i, row in jeopardy.iterrows():
        # Split the clean_question column of the row on the space character (), and assign to split_question 
        split_question = row["clean_question"].split(" ")
        # Remove any words in split_question that are less than 5 characters long
        split_question = [q for q in split_question if len(q) > 4]
        # Set match_count to 0
        match_count = 0
        # Loop through each word in split_question 
        for word in split_question:
            # If the term occurs in terms_used, add 1 to match_count
            if word in terms_used:
                match_count += 1
        # Add each word in split_question to terms_used using the add method on sets 
        for word in split_question:
            terms_used.add(word)
        # If the length of split_question is greater than 0, divide match_count by the length of split_question
        if len(split_question) > 0:
            match_count /= len(split_question)
        # Append match_count to question_overlap
        question_overlap.append(match_count)

# Assign question_overlap to the question_overlap column of jeopardy 
jeopardy["question_overlap"] = question_overlap
# Find the mean of the question_overlap column and print it 
jeopardy["question_overlap"].mean()
```




    0.7513452152538761



There is around 75% overlap between terms in new questions and terms in old questions. Our function has given us some insight that it might be worth looking further into the recycling of questions based on phrases as opposed to single terms.

## 4.3 Low Value vs High Value Questions

Let's say we only want to study questions that pertain to high value questions instead of low value questions. We can figure out which terms correspond to high-value questions using a **chi-squared test**.  

First we need to narrow down the questions into two categories:

- Low value: Any row where Value is less than 800
- High value: Any row where Value is greater than 800


```python
# If the clean_value column is greater than 800, assign 1 to value, otherwise assign 0 to value
def determine_value(row):
    value = 0
    if row["clean_value"] > 800:
        value = 1
    return value

jeopardy["high_value"] = jeopardy.apply(determine_value, axis=1)
```


```python
jeopardy.head(2)
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Show Number</th>
      <th>Air Date</th>
      <th>Round</th>
      <th>Category</th>
      <th>Value</th>
      <th>Question</th>
      <th>Answer</th>
      <th>clean_question</th>
      <th>clean_answer</th>
      <th>clean_value</th>
      <th>answer_in_question</th>
      <th>question_overlap</th>
      <th>high_value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4680</td>
      <td>2004-12-31</td>
      <td>Jeopardy!</td>
      <td>HISTORY</td>
      <td>$200</td>
      <td>For the last 8 years of his life, Galileo was ...</td>
      <td>Copernicus</td>
      <td>for the last 8 years of his life galileo was u...</td>
      <td>copernicus</td>
      <td>200</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4680</td>
      <td>2004-12-31</td>
      <td>Jeopardy!</td>
      <td>ESPN's TOP 10 ALL-TIME ATHLETES</td>
      <td>$200</td>
      <td>No. 2: 1912 Olympian; football star at Carlisl...</td>
      <td>Jim Thorpe</td>
      <td>no 2 1912 olympian football star at carlisle i...</td>
      <td>jim thorpe</td>
      <td>200</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



We'll be able to reuse our `term_used` set to loop through each of the terms and:

- Find the number of low value questions the word occurs in
- Find the number of high value questions the word occurs in
- Find the percentage of questions the word occurs in
- Find counts, based on the percentage of questions the word occurs in
- Compute the chi squared value based on the expected counts and the observed counts for high and low value questions


```python
def count_usage(term):
    low_count = 0
    high_count = 0
    for i, row in jeopardy.iterrows():
        if term in row["clean_question"].split(" "):
            if row["high_value"] == 1:
                high_count += 1
            else:
                low_count += 1
    return high_count, low_count

comparison_terms = list(terms_used)[:5]
observed_expected = []
for term in comparison_terms:
    observed_expected.append(count_usage(term))

observed_expected
```




    [(0, 2), (1, 0), (0, 1), (0, 1), (1, 0)]



We can now find the words with the biggest differences in usage between high and low value questions, by selecting the words with the highest associated chi-squared values.

# 5. Applying the Chi-Squared Test

Computing the expected counts and the chi-squared value.


```python
import numpy as np
from scipy.stats import chisquare

# Find the number of rows in jeopardy where high_value is 1, and assign to high_value_count
high_value_count = jeopardy[jeopardy["high_value"] == 1].shape[0]
# Find the number of rows in jeopardy where high_value is 0, and assign to low_value_count
low_value_count = jeopardy[jeopardy["high_value"] == 0].shape[0]

# Create an empty list called chi_squared
chi_squared = []
# Loop through each list in observed_expected
for obs in observed_expected:
    # Add up both items in the list (high and low counts) to get the total count, and assign to total
    total = sum(obs)
    # Divide total by the number of rows in jeopardy to get the proportion across the dataset. Assign to total_prop
    total_prop = total / jeopardy.shape[0]
    # Multiply total_prop by high_value_count to get the expected term count for high value rows
    high_value_exp = total_prop * high_value_count
    # Multiply total_prop by low_value_count to get the expected term count for low value rows
    low_value_exp = total_prop * low_value_count
    # Use the scipy.stats.chisquare function to compute the chi-squared value and p-value given the expected and observed counts
    observed = np.array([obs[0], obs[1]])
    expected = np.array([high_value_exp, low_value_exp])
    # Append the results to chi_squared
    chi_squared.append(chisquare(observed, expected))

chi_squared
```




    [Power_divergenceResult(statistic=2.4877921171956752, pvalue=0.11473257634454047),
     Power_divergenceResult(statistic=0.40196284612688399, pvalue=0.52607729857054686),
     Power_divergenceResult(statistic=0.80392569225376798, pvalue=0.36992223780795708),
     Power_divergenceResult(statistic=2.4877921171956752, pvalue=0.11473257634454047),
     Power_divergenceResult(statistic=2.4877921171956752, pvalue=0.11473257634454047)]



None of the terms had a significant difference in usage between high value and low value rows. Additionally, the frequencies were all lower than 5, so the chi-squared test isn't as valid. It would be better to run this test with only terms that have higher frequencies.

# 6. Recommendations

**1. Find a better way to eliminate non-informative words than just removing words that are less than 5 characters long. For example:**  
- Manually create a list of words to remove, like the, than, etc
- Find a list of stopwords to remove
- Remove words that occur in more than a certain percentage (like 5%) of questions  

**2. Perform the chi-squared test across more terms to see what terms have larger differences.**  
- Use the apply method to make the code that calculates frequencies more efficient
- Only select terms that have high frequencies across the dataset, and ignore the others

**3. Look more into the Category column and see if any interesting analysis can be done with it.**  
- See which categories appear the most often
- Find the probability of each category appearing in each round
- Use phrases instead of single words when seeing if there's overlap between questions. Single words don't capture the whole context of the question well
