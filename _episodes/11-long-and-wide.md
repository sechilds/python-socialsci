---
title: "Wide and long data formats"
teaching: 20
exercises: 15
questions:
- "What are long and Wide formats?"
- "Why would I want to change between them?"
objectives:
- "Explain difference between long and wide foramts and why each might be used"
- "Illustrate how to change between formats using the `melt` and `pivot` methods"
keypoints:
- "The `melt` method can be used to change from wide to long format"
- "The `pivot` method can be used to change from the long to wide format"
- "Aggregations are best done from data in the long format."
---

## Wide and long data formats

In the SN7577 dataset that we have been using there is a group of columns which record which daily newspapers each respondent reads. Despite the un-informative names like 'daily1' each column refers to a current UK daily national or local newspaper. 

Whether the paper is read or not is recorded using the values of 0 or 1 as a boolean indicator. The advantage of using a column for each paper means that should a respondent read multiple newpapers, all of the required information can still be recorded in a single record.

Recording information in this 'wide' format is not always beneficial when trying to analyse the data.

Pandas provides methods for converting data from 'wide' to 'long' format and from 'long' to 'wide' format

The SN7577 dataset does not contain a variable that can be used to uniquely identify a row. This is often referred to as a 'primary key' field (or column).

A dataset doesn't need to have such a key. None of the work we have done so far has required it. 

When we create a pandas dataframe by importing a csv file, we have seen that pandas will create an index  for the rows. This index can be used a bit like a key field, but as we have seen there can be problems with the index when we concatenate two dataframes together.

In the version of SN7577 that we are going to use to demonstrate long and wide formats we will add a new variable with the name of 'Id' and we will restrict the other columns to those starting with the word 'daily'.

~~~
import pandas as pd
df_SN7577 = pd.read_csv("SN7577.tab", sep='\t')
~~~
{: .language_python}

We will create a new dataframe with a single column of 'Id'. 

~~~
# create an 'Id' column
df_papers1 = pd.DataFrame(pd.Series(range(1,1287)),index=None,columns=['Id'])
~~~
{: .language_python}

Using the range function I can create values of Id starting with 1 and going upto 1286 (remember the second parameter to range is one past the last value used.) I have explicitly coded this value because I knew how many rows were in the dataset. If I didn't, I could have used 

~~~
len(df_SN7577.index) +1
~~~
{: .language_python}

We will create a 2nd dataframe, based on SN7577 but containing only the columns starting with the word 'daily'. 

There are several ways odf doing this.

we could use the `iloc` method and provide the index values of the range of columns we want.

~~~
df_papers2 = df_SN7577.iloc[:,118:143]
~~~
{: .language_python}

This isn't really very practical. I would need to know the position of all of the columns of interest. They may not be contiguous. This method would be very prone to human error.

We could use a regular expression. Regular expressions is a very complex topic which we won't be covering. Using the `filter` method the code

~~~
df_papers2 = df_SN7577.filter(regex= '^daily')
~~~
{: .language_python}

will do want we want. '^daily' is a simple regular exprerssion which says 'startswith' the characters 'daily'

A simpler way is to use the 'like' parameter of the `filter` method.

~~~
df_papers2 = df_SN7577.filter(like= 'daily')
~~~
{: .language_python}

The value supplied to 'like' can occur anywhere in the column name to be matched (and therefore selected)

To create the dataframe that we will use, we will concatenate the two dataframes we have created. 

~~~
df_papers = pd.concat([df_papers1, df_papers2], axis = 1)
print(df_papers.index)
print(df_papers.columns)
~~~
{: .language_python}

We use 'axis = 1' because we are joining by columns not rows which is the default.


## From 'wide' to 'long'

Just to make the displays more maneagable we will use only the first eight 'daily' columns

~~~
## using df_papers
daily_list = ['daily1', 'daily2','daily3', 'daily4','daily5', 'daily6','daily7', 'daily8']

df_daily_papers_long = pd.melt(df_papers, id_vars = ['Id'], value_vars = daily_list)

# by default the new columns created will be called 'variable' which is the name of the 'daily'
# and 'value' which is the value of that 'daily' for that 'Id'. So we will rename the columns

df_daily_papers_long.columns = ['Id','Daily_paper','Value']
df_daily_papers_long
~~~
{: .language_python}

We now have a dataframe that we can `groupby`. 

We want to `groupby` the 'Daily_paper' and them sum the 'Value'.

~~~
a = df_daily_papers_long.groupby('Daily_paper')['Value'].sum()
a
~~~
{: .language_python}

## From Long to Wide 

The process can be reversed by using the `pivot` method. 
Here we need to indicate which column (or columns) remain fixed (this will become an index in the new dataframe), which column contains the values which are to become column names and which column contains the values for the columns.

In our case we want to use the 'Id' column as the fixed column, the 'Daily_paper' column contains the column names and the 'Value' column contains the values.

~~~
df_daily_papers_wide = df_daily_papers_long.pivot(index = 'Id', columns = 'Daily_paper', values = 'Value')
~~~
{: .language_python}

We can change our 'Id' index back to an ordinary column with 

~~~
df_daily_papers_wide.reset_index(level=0, inplace=True)
~~~
{: .language_python}

> ## Exercise
> 
> 1. Find out how many people take each of the daily newspapers by Title.
> 2. Which titles don't appear to be read by anyone?
> 
> There is a file called Newspapers.csv which lists all of the newpapers Titles along with the corresponding 'daily' value
> 
> Help : Newspapers.csv cotains both daily and Sunday newspapers. you can filter out the Sunday papers with the following code;
> 
> 
> ~~~
> df_newspapers = df_newspapers[(df_newspapers.Column_name.str.startswith('daily'))]
> ~~~
> {: .language_python}
> 
> > ## Solution
> > 
> > 1. Read in Newspapers.csv file and keep only the dailies.
> > 
> > ~~~
> > df_newspapers = pd.read_csv("Newspapers.csv")
> > df_newspapers = df_newspapers[(df_newspapers.Column_name.str.startswith('daily'))]
> > df_newspapers
> > ~~~
> > {: .language_python}
> > 
> > 2. Create the df_papers dataframe as we did before.
> > 
> > ~~~
> > import pandas as pd
> > df_SN7577 = pd.read_csv("SN7577.tab", sep='\t')
> > #create an 'Id' column
> > df_papers1 = pd.DataFrame(pd.Series(range(1,1287)),index=None,columns=['Id'])
> > df_papers2 = df_SN7577.filter(like= 'daily')
> > df_papers = pd.concat([df_papers1, df_papers2], axis = 1)
> > df_papers
> > ~~~~
> > {: .language_python}
> > 
> > 3. Create a list of all of the dailies, one way would be
> > 
> > ~~~
> > daily_list = []
> > for i in range(1,26):
> >     daily_list.append('daily'+str(i))  
> > ~~~
> > {: .language_python}
> > 
> > 4. Pass the list as the 'value_vars' parameter to the `melt` method
> > 
> > 
> > ~~~
> > #use melt to create df_daily_papers_long  
> > df_daily_papers_long = pd.melt(df_papers, id_vars = ['Id'], value_vars = daily_list )
> > #Change the column names
> > df_daily_papers_long.columns = ['Id','Daily_paper','Value']
> > ~~~
> > {: .language_python}
> > 
> > 5. `merge` the two dataframes with a left join, because we want all of the Newspaper Titles to be included.
> > 
> > ~~~
> > df_papers_taken = pd.merge(df_newspapers, df_daily_papers_long, how='left', left_on = 'Column_name',right_on = 'Daily_paper')
> > ~~~
> > {: .language_python}
> > 
> > 6. Then `groupby` the 'Title' and sum the 'Value'
> > 
> > ~~~
> > df_papers_taken.groupby('Title')['Value'].sum()
> > ~~~
> > {: .language_python}
> {: .solution}
{: .challenge}
