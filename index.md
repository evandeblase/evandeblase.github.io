---
layout: wide_default
---    
## About Me

Hi, my name is Evan DeBlase. I am a senior at Lehigh University studying finance and accounting. I have particular interest in real estate, anything from the financial aspect of valuation to practical development. I am graduating in May planning to begin my career in accounting with SEI. 

<!-- Upload your own photo and change the path -->

---

## Portfolio

<!-- You can link to other websites, PDFs in this repo, and other pages in this repo -->

_**[10K Sentiment and its Immediate Affect on Return](report)**_

# ASSIGNMENT 5: TEXTUAL ANALYSIS FOR 10-K REPORTS
## Summary
&nbsp;&nbsp;&nbsp;&nbsp;The question for this analysis is, does the sentiment of a 10-K filing have any immediate (within 10 days) affect on the company's stock performance? To conduct this analysis, I calculated positive and negative sentiment scores for each 10-K for firms in the S&P 500 and compared those scores to the stock return. <br>
&nbsp;&nbsp;&nbsp;&nbsp;The results of this study are difficult to understand for me, however I found that sentiment, whether positive or negative, has little effect on return around the filing date of a 10-K. The correlations of each variable are low, and the data is largely situated in a single cluster. Sentiment scores varied across the different dictionaries, but there is not much difference in return compared to the scores.
## 2. Data
The sample used for this analysis was the most recent 10-K filing from firms in the S&P 500 as of December 2022. Along with these filings, the daily returns for dates around the filing are used to compare the sentiment of the text to actual stock performance. More specifically the return on the day of the filing up to 10 days after.

The return variables are built by taking the filing date and the next 10 dates after it and finding cumulative return.
- `v1` is a the return on the date of the filing
- `v2` is a the cumulative return on the date of the filing and the next two days
- `v3` is a the cumulative return on the date of the filing and the next nine days
The following is the process of creating return variables, explained by the comments
```python
# First: calculated the cumulative product of the return variables. ret_1 is ret + 1
firm_ret_data = firm_ret_data.assign(cum_ret = lambda x: x.groupby(['ticker'])['ret_1'].cumprod() - 1)

# Second: Find return where filing_date == date
matching_rows = firm_ret_data[firm_ret_data['filing_date'] == firm_ret_data['date']]

isolated_indices = []

# Third: get the returns for the v2 and v3 variables. This searches the index for the dates we want.
for index in matching_rows.index:
    if index in firm_ret_data.index:
        isolated_indices.append(index)
    if index + 2 in firm_ret_data.index:
        isolated_indices.append(index + 2)
    if index + 9 in firm_ret_data.index:
        isolated_indices.append(index + 9)

# Last: Add the cumulative returns to a new dataframe for only the selected dates.
firm_ret_data = firm_ret_data.loc[isolated_indices]
firm_ret_data_final = firm_ret_data[['ticker','date','cum_ret']]
```

The sentiment variables are essentially the number of words from given dictionaries divided by the total length of the document. These variables act as an index that measures the general tone of the 10-K report. The following is an example of how one of the variables was created, explained by the comments.
```python
# First: Load and read the dictionary. This creates a dataframe that consists of all the words from LM_MasterDictionary_1993-2021
LM_dict = open('inputs\\LM_MasterDictionary_1993-2021.csv', 'r')
LM_dict = pd.read_csv(LM_dict, index_col=0)

# Second: To make navigating the dataframe simpler, reset the index to 'Word'. 
LM_dict = LM_dict.reset_index().rename(columns = {'index':'Word'})

# Third: Query in the dictionary for any word that has a score of non-zero in the Positive column. The same process was followed for the negative column, as well.
LM_positive = LM_dict.query('Positive != 0')

# Fourth: Format the positive words to fit into NEAR_regex, which will be discussed later.
LM_positive = LM_positive['Word'].to_list() # Convert to a list
LM_positive = map(lambda x:x.lower(), LM_positive) # Convert all words to lowercase
LM_positive = list(LM_positive) # Convert to a list again
LM_pos = "("+ "|".join(LM_positive) +")" # Add the necessary formatting for NEAR_Regex --> ('word1'|'word2'|'word3')

# Last: Get the number of words from the dictionary and divide by the document length. Multiplied by 100 to convert to a percent.
'LM_positive': (len(re.findall(NEAR_regex([LM_pos]), document)) / len(document)) * 100
```
The same process was followed for the three topics measured in variables 5-10 with a slight twist. NEAR_regex searched for the words from BHR_positive and BHR_negative (the other dictionaries used) relative to the three topics I chose.

### Sentiment Variables
1. LM_positive = 354 words
2. LM_negative = 2355 words
3. (ML)BHR_positive = 75 words
4. (ML)BHR_negative = 94 words

### NEAR_regex
Two important inputs to this function are ```partial``` and ```max_words_between```. I chose not to add ```partial``` to my analysis and instead tried to accomodate every possible variation of a specific word in the topic. For example if 'regulation' was a word, I would include: ```'regulate', 'regulated', 'regulation'```, and so forth. I do notice the shortcomings of this approach due to the limits of my brain and google, however I believe I created a thorough enough list.
I chose to increase the amount of ```max_words_between``` to 20. The justification for this is as simple as the average words in a business-setting sentence. I found this just through a quick Google search.

### Contextual Topics
1. Government Regulation
    - Government Regulation is a topic that affects every business in a variety of ways. With the ever-changing world, it's the government's responsibility to protect the interests of its people. One way they achieve this is through regulating trade, whether or not that's the way to do it is a story for another day. However, government regulation has very real effects on a firm's performance, which is why I believe its discussion in a 10-K report is important.
2. Financing Terms
    - The terms of a business's financing needs is favorable, unfavorable, or somewhere in between. This list consists of any word, again that I could think of, related to stock or bond issuances. New financing is something most companies go through yearly, and existing financing is discussed in great detail in multiple places throughout the report. I believe the sentiment around financing is a good indicator for the textual analysis.
3. Litigation
    - Lawsuits are something that most businesses face, and especially so in the top 500 U.S. companies. Everyone wants a piece of their $$$. This list consists of any legal terms related to the conduct of business. Litigation can cause huge losses for firms, and while it is usually spoken about negatively, the confidence of the firm's ability to win or lessen the impact of the outcome is important. While litigation is usually not analysts' first concern, it will have an impact on the business. Bias for the negative variable is possible, and a way around that would to only count negative or positive words, excluding where they mention both. 
### Summary Stats
Above are the summary statistics for the textual analysis sample.
- Count is 501, which is a shortcoming of my analysis. I managed to get 498 total 10-K reports from 503 total firms, and the count here is higher than what is expected. 
- Comparing the mean and median values of each variable can help figure out if there are outliers in the dataset; their values should be similar to each other. There doesn't appear to be any outliers, meaning no 10-K is overly positive or negative (I wonder if that's intentional by the business). 
- The statistics of the first four variables are naturally higher than the last six because they do not compare one list to another, they simply just search for positive or negative words.
- The statistics of LM_positive are much lower compared to its brothers and sisters. I believe this is because of the large difference in wordcount between LM_positive and LM_negative.
### Contextual Sentiment
- One thing I'm noticing is that there are more negative words in both dictionaries. The LM dictionary was not used for these variables so their difference doesn't matter, however there are 19 (25%) more negative words in the dictionary that was used. This means generally the negative scores should be higher. The summary data shows this is generally the case. This can possibly skew the data in the negative direction.
- Overall, these contextual variables all follow the same process of being created, whether or not it is a robust way to measure sentiment would cause for further analysis on the dictionaries I used.

#### Charts

Compare / contrast the relationship between the returns variable and the two “LM Sentiment” variables (positive and negative) with the relationship between the returns variable and the two “ML Sentiment” variables (positive and negative). Focus on the patterns of the signs of the relationships and the magnitudes.
- The patterns of both LM and ML charts are very similar, however the machine learning dictionary has comparatively higher sentiment scores consistent with both positive and negative, attributed to the differences in the sizes of the dictionaries. We would expect a positive correlation for the positive sentiment charts and a negative correlation for the negative sentiment charts. The data suggests, with the low correlations and no discernable pattern, neither positive nor negative sentiment scores have an effect on return. While it is difficult to say the sentiment has **NO** effect on return, there are many other factors that go into stock pricing. To more accurately predict returns we must consider those variables as well.

If your comparison/contrast conflicts with Table 3 of the Garcia, Hu, and Rohrer paper (ML_JFE.pdf, in the repo), discuss and brainstorm possible reasons why you think the results may differ. If your patterns agree, discuss why you think they bothered to include so many more firms and years and additional controls in their study? (It was more work than we did on this midterm, so why do it to get to the same point?)
- Our results are similar, not much difference in return compared to sentiment. They included much more data because they had the means to, and a larger sample size is always beneficial in a statistical analysis. More firms also allows for analysis based on the growth of an industry, for example. More years of return data also factors in macroeconomic factors that have an effect on the market as a whole. The analysis I conducted for this report only considers 2022, which is a small sample dictated by the direction of the market.

Discuss your 3 “contextual” sentiment measures. Do they have a relationship with returns that looks “different enough” from zero to investigate further? If so, make an economic argument for why sentiment in that context can be value relevant.
- For each different topic, I notice a relationship that the highest and lowest returns have similar sentiment scores across the board, for both positive and negative. This is indicated by an implied vertical line near the highest and lowest returns.

Is there a difference in the sign and magnitude? Speculate on why or why not.
- I am not noticing a difference between sign and magnitude. This makes sense because the data is so loosely correlated and there seems to be a pretty even distribution of positive and negative returns.

<!-- <img src="images/dummy_thumbnail.jpg?raw=true"/> -->

---

_**[Regression Practice](Regression_practice)**_

Or: The process that created this page can be used to show off your whole midterm analysis file, as is.

<!-- <img src="images/dummy_thumbnail.jpg?raw=true"/> -->

---

_**[Eventual team project]([https://donbowen.github.io/teamproject/](https://github.com/adrianmross/congress_trades_dashboard))**_
Our team project tracks the portfolios of congress members and their returns. We estimate their portfolio value based on their buys and sells of stocks because the information of their portfolios is not publicly available. We also flag trades for insider trading, based on whether or not the sit on a committee that directly influences the legislation of a stock they buy.

<img src="images/dummy_thumbnail.jpg?raw=true"/>

---

_**[Some personal project](/pdf/OldBridge_Valuation_Report_REAL348.pdf)**_

<img src="images/cover_page.pdf"/>

---

## Career Objectives

The end-goal of my career is to say I was able to help others while staying focused on what interests me. My interests are in real estate as mentioned in my about me section, and a core value I live by is selflessness. 
<br>
A place to live is a universal need and I want to be able to provide that for people. I plan to break into this industry once I have built enough capital and relationships to provide solidarity in my endeavors. Real estate is not an easy market without experience and money!

---

## Hobbies

I enjoy everything adventurous and experiencing new things. Whether it be new ideas, new places, or pretty much new anything, I love learning and challenging myself to new things.
<br>
To be more specific about my hobbies, I enjoy anything outdoors; from hunting and fishing to hiking and camping. A fun fact about me is that I took a car-camping road trip from my home state of PA all the way to San Diego... and back! I also enjoy building things, most notably the card table my roommate and I built in the woodshop at Lehigh. I don't have many projects to my name, but I am working on a bookshelf currently.
<br>
Like most people, I value my time with friends and family, so I have to include them somewhere on my site. Any other hobbies I previously mentioned were things both my friends and family helped me enjoy doing!

---
<p style="font-size:11px">Page template forked from <a href="https://github.com/evanca/quick-portfolio">evanca</a></p>
<!-- Remove above link if you don't want to attibute -->
