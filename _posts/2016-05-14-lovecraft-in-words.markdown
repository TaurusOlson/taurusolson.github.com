---
layout: post
title:  "Lovecraft in words"
date:   2016-05-14
categories: [python, data]
---


In one of the Java courses I took on Coursera, a simple exercise consists in counting the occurrences of the words in a file. The examples we used were plays from Shakespeare and while I've read a few of them I prefer Lovecraft and Python. 

The idea is simple: 

* we read a file
* split it into words
* count the occurrences of the words that are not common english words.

We'll repeat this operation for many short stories of Lovecraft and we'll visualize the most frequent words of each short story. Hopefully, this should give us an idea of what each one is about.

This post was created using the [Jupyter Notebook](http://jupyter.org/) with a few libraries:


{% highlight python %}
%matplotlib inline
import os
from collections import defaultdict
import time
import re
from fntools import count
import matplotlib.pyplot as plt
import pandas as pd
import matplotlib
matplotlib.style.use('ggplot')
from wordcloud import WordCloud
from itertools import repeat
{% endhighlight %}

## Counting the relevant words 

Here are the functions we'll need to count the words.


{% highlight python %}
def count_words_frequency(inputfile, excluded=[]):
    """Count the frequency of the words in the inputfile
    
    Excluded words can be specified with the keywords `excluded`.
    
    """
    frequencies = defaultdict(int)
    with open(inputfile) as f:
        for line in f:
            for word in line.split():
                word = word.lower().strip()
                word = re.sub("[^a-z]", "", word)
                if len(word) > 0 and word not in excluded:
                    frequencies[word] += 1
    return frequencies


def count_total_words(frequencies):
    """Count the total number of words from the given frequency table"""
    return sum(count for word, count in frequencies.items())

{% endhighlight %}

In order to exclude the words that occur frequently in the english language and that would distort the results I used the 10000 most common english words as determined by [n-gram frequency analysis of the Google's Trillion Word Corpus](https://github.com/first20hours/google-10000-english).


{% highlight python %}
CURRENT_DIR = os.path.expanduser("~/Dropbox/Projects/Lovecraft")
ENGLISH_WORDS_FILE = CURRENT_DIR + "/data/google-10000-english.txt"

def get_most_common_english_words(max_word_count=1000):
    """Return a list of the most common english words.
    
    The maximum number of words can be specified with `max_word_count`.
    The maximum number of available words is 10000.
    
    """
    most_common_english_words = []
    n = 0
    with open(ENGLISH_WORDS_FILE) as f:
        while n < max_word_count:
            most_common_english_words.append(f.readline().strip())
            n += 1
    return most_common_english_words
{% endhighlight %}

Now the processing part. There's a bit of calibration to do to determine the most common english words we consider. Indeed if we exclude the 10000 most common words, we'll have mainly the names of the protagonists, or rare words left. In some cases, this may not tell us much about what is happening but rather who is in the story. On the other hand if we choose a lower value we may get some useful words but other may be irrelevant.
After some tests, I chose to exclude the first 5000 most common words. Note that we could adapt this value to each short story but, for the sake of simplicity, we'll stick to 5000 for the 6 texts we'll analyze.

Then let's apply our function for counting the frequency of the words (`count_words_frequency`) and store them in a `pandas.DataFrame`:


{% highlight python %}
filenames = ("cthulhu.txt", "mountains_of_madness.txt", "the_unnamable.txt", "charles_dexter_ward.txt", 
         "wall_of_sleep.txt", "erich_zann.txt")

english_words = get_most_common_english_words(5000)

words_frequency = {}
total_words = {}
dfs = {}

for filename in filenames:
    inputfile = os.path.join(CURRENT_DIR, "data", filename)
    name = filename[:-4]
    print 'Reading %s...' % filename
    words_frequency[name] = count_words_frequency(inputfile, english_words)
    total_words[name] = count_total_words(words_frequency[name])
    dfs[name] = pd.DataFrame(words_frequency[name].items(), columns=('word', 'frequency'))
    
{% endhighlight %}

<!--
Reading cthulhu.txt...
Reading mountains_of_madness.txt...
Reading the_unnamable.txt...
Reading charles_dexter_ward.txt...
Reading wall_of_sleep.txt...
Reading erich_zann.txt...
-->



Let's look at the example of 'The Call of Cthulhu' and see what the 3 most frequent words are:


{% highlight python %}
dfs['cthulhu'].sort_values(by='frequency', ascending=False).head(3)
{% endhighlight %}



<p>
<table align="center">
  <thead>
    <tr>
      <th>word</th>
      <th>frequency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>cult</td>
      <td>30</td>
    </tr>
    <tr>
      <td>johansen</td>
      <td>23</td>
    </tr>
    <tr>
      <td>cthulhu</td>
      <td>22</td>
    </tr>
  </tbody>
</table>
</p>



From what we see, 'The Call of Cthulhu' is mainly about a cult, a person called Johansen and the famous Cthulhu. Sounds right!

## Visualization

We visualize the most frequent words with a word cloud using the [word_cloud](https://github.com/amueller/word_cloud) package:


{% highlight python %}
def plot_word_cloud(data, n=20):
    """Plot the n most frequent words with wordcloud"""
    text = []
    for word, freq in data:
        text.extend(list(repeat(word, freq)))

    text = ','.join(text)
    # Generate a word cloud image
    wordcloud = WordCloud(background_color='white').generate(text)

    # Display the generated image:
    # the matplotlib way
    plt.figure(figsize=(8, 6))
    plt.imshow(wordcloud)
    plt.axis("off")
    return plt
{% endhighlight %}


### The call of Cthulhu


{% highlight python %}
cthulhu = dfs['cthulhu'].sort_values(by='frequency', ascending=False).head(10)
cthulhu
{% endhighlight %}



<p>
<table align="center">
  <thead>
    <tr>
      <th>word</th>
      <th>frequency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>cult</td>
      <td>30</td>
    </tr>
    <tr>
      <td>johansen</td>
      <td>23</td>
    </tr>
    <tr>
      <td>cthulhu</td>
      <td>22</td>
    </tr>
    <tr>
      <td>uncle</td>
      <td>21</td>
    </tr>
    <tr>
      <td>legrasse</td>
      <td>20</td>
    </tr>
    <tr>
      <td>wilcox</td>
      <td>19</td>
    </tr>
    <tr>
      <td>angell</td>
      <td>13</td>
    </tr>
    <tr>
      <td>whilst</td>
      <td>12</td>
    </tr>
    <tr>
      <td>manuscript</td>
      <td>11</td>
    </tr>
    <tr>
      <td>basrelief</td>
      <td>11</td>
    </tr>
  </tbody>
</table>
</p>



{% highlight python %}
plot_word_cloud(cthulhu.values);
{% endhighlight %}


![png](/assets/lovecraft/cthulhu.png)


### The mountains of madness


{% highlight python %}
mountains_of_madness = dfs['mountains_of_madness'].sort_values(by='frequency', ascending=False).head(10)
mountains_of_madness
{% endhighlight %}



<p>
<table align="center">
  <thead>
    <tr>
      <th>word</th>
      <th>frequency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>danforth</td>
      <td>53</td>
    </tr>
    <tr>
      <td>antarctic</td>
      <td>48</td>
    </tr>
    <tr>
      <td>vast</td>
      <td>43</td>
    </tr>
    <tr>
      <td>sculptures</td>
      <td>41</td>
    </tr>
    <tr>
      <td>specimens</td>
      <td>37</td>
    </tr>
    <tr>
      <td>monstrous</td>
      <td>31</td>
    </tr>
    <tr>
      <td>curious</td>
      <td>31</td>
    </tr>
    <tr>
      <td>carvings</td>
      <td>30</td>
    </tr>
    <tr>
      <td>abyss</td>
      <td>29</td>
    </tr>
    <tr>
      <td>peaks</td>
      <td>28</td>
    </tr>
  </tbody>
</table>
</p>



{% highlight python %}
plot_word_cloud(mountains_of_madness.values);
{% endhighlight %}


![png](/assets/lovecraft/mountains_of_madness.png)


### The unnamable


{% highlight python %}
the_unnamable = dfs['the_unnamable'].sort_values(by='frequency', ascending=False).head(10)
the_unnamable
{% endhighlight %}



<p>
<table align="center">
  <thead>
    <tr>
      <th>word</th>
      <th>frequency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>manton</td>
      <td>13</td>
    </tr>
    <tr>
      <td>unnamable</td>
      <td>9</td>
    </tr>
    <tr>
      <td>attic</td>
      <td>9</td>
    </tr>
    <tr>
      <td>tomb</td>
      <td>7</td>
    </tr>
    <tr>
      <td>deserted</td>
      <td>7</td>
    </tr>
    <tr>
      <td>whispered</td>
      <td>5</td>
    </tr>
    <tr>
      <td>slab</td>
      <td>4</td>
    </tr>
    <tr>
      <td>spectral</td>
      <td>4</td>
    </tr>
    <tr>
      <td>legends</td>
      <td>3</td>
    </tr>
    <tr>
      <td>slate</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</p>



{% highlight python %}
plot_word_cloud(the_unnamable.values);
{% endhighlight %}


![png](/assets/lovecraft/the_unnamable.png)


### The case of Charles Dexter Ward


{% highlight python %}
charles_dexter_ward = dfs['charles_dexter_ward'].sort_values(by='frequency', ascending=False).head(10)
charles_dexter_ward
{% endhighlight %}




<p><table align="center">
  <thead>
    <tr>
      <th>word</th>
      <th>frequency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>willett</td>
      <td>169</td>
    </tr>
    <tr>
      <td>curwen</td>
      <td>158</td>
    </tr>
    <tr>
      <td>ye</td>
      <td>87</td>
    </tr>
    <tr>
      <td>wards</td>
      <td>52</td>
    </tr>
    <tr>
      <td>pawtuxet</td>
      <td>50</td>
    </tr>
    <tr>
      <td>curwens</td>
      <td>43</td>
    </tr>
    <tr>
      <td>providence</td>
      <td>37</td>
    </tr>
    <tr>
      <td>weeden</td>
      <td>35</td>
    </tr>
    <tr>
      <td>capt</td>
      <td>33</td>
    </tr>
    <tr>
      <td>curious</td>
      <td>33</td>
    </tr>
  </tbody>
</table>
</p>



{% highlight python %}
plot_word_cloud(charles_dexter_ward.values);
{% endhighlight %}


![png](/assets/lovecraft/charles_dexter_ward.png)


### Beyond the wall of sleep


{% highlight python %}
wall_of_sleep = dfs['wall_of_sleep'].sort_values(by='frequency', ascending=False).head(10)
wall_of_sleep
{% endhighlight %}




<p><table align="center">
  <thead>
    <tr>
      <th>word</th>
      <th>frequency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>slater</td>
      <td>25</td>
    </tr>
    <tr>
      <td>waking</td>
      <td>6</td>
    </tr>
    <tr>
      <td>decadent</td>
      <td>5</td>
    </tr>
    <tr>
      <td>cosmic</td>
      <td>5</td>
    </tr>
    <tr>
      <td>cannot</td>
      <td>4</td>
    </tr>
    <tr>
      <td>couch</td>
      <td>4</td>
    </tr>
    <tr>
      <td>ethereal</td>
      <td>4</td>
    </tr>
    <tr>
      <td>valleys</td>
      <td>4</td>
    </tr>
    <tr>
      <td>oppressor</td>
      <td>4</td>
    </tr>
    <tr>
      <td>luminous</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</p>



{% highlight python %}
plot_word_cloud(wall_of_sleep.values);
{% endhighlight %}


![png](/assets/lovecraft/wall_of_sleep.png)


### The music of Erich Zann


{% highlight python %}
erich_zann = dfs['erich_zann'].sort_values(by='frequency', ascending=False).head(10)
erich_zann
{% endhighlight %}




<p>
<table align="center">
  <thead>
    <tr>
      <th>word</th>
      <th>frequency</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>zann</td>
      <td>17</td>
    </tr>
    <tr>
      <td>viol</td>
      <td>13</td>
    </tr>
    <tr>
      <td>rue</td>
      <td>11</td>
    </tr>
    <tr>
      <td>dauseil</td>
      <td>11</td>
    </tr>
    <tr>
      <td>garret</td>
      <td>8</td>
    </tr>
    <tr>
      <td>erich</td>
      <td>8</td>
    </tr>
    <tr>
      <td>dumb</td>
      <td>6</td>
    </tr>
    <tr>
      <td>zanns</td>
      <td>5</td>
    </tr>
    <tr>
      <td>strains</td>
      <td>4</td>
    </tr>
    <tr>
      <td>shutter</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</p>



{% highlight python %}
plot_word_cloud(erich_zann.values);
{% endhighlight %}


![png](/assets/lovecraft/erich_zann.png)


## Conclusion

From the results, we clearly see the topics Lovecraft treated in his stories: strange cults, dreams, madness and powerless men facing terrifying creatures.
From a technical point of view, this approach is a bit simplistic but it's a good approximation.
This study could be improved with natural language processing. Instead of creating a meaning from a list of words, we could analyze the sentiments in the each short story.


