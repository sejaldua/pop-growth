# pop-growth
This project was all about learning how to animate graphs using matplotlib and seaborn. Just enter a city and the script will generate what I call a "snake graph" depicting population growth over time.

Hours spent: 20

Topics explored:
- `matplotlib`
- `seaborn`
- `pandas` dataframes
- web scraping
- data cleaning
- data visualization
- animation


My inspiration to start this project came from a really interesting Medium article I read a while back. It was called [**How to Create Animated Graphs in Python**](https://towardsdatascience.com/how-to-create-animated-graphs-in-python-bb619cc2dec1 "Medium Article"). The article provided me with an introductory education of `Matplotlib` and `Seaborn` libraries. I knew that I wanted to do away with static graphs for this one because it's time to explore the next level of data vis. The example the article provided me with pertained to the opioid crisis in the USA. The plot that the author produced was an animation of number of heroin deaths by overdose in the USA over time. I have recreated that example in the [heroin overdoses file](heroin_overdoses_example.ipynb). 

Below is the animation as a GIF file:
![heroin overdoses gif](HeroinOverdosesAugmented.gif)

I wanted to visualize something a little more light-hearted, and I also wanted to acquire the data with minimal effort. I did not want to download a massive Excel file and then use Python to extract the data from the cells and then create a Pandas dataframe. The alternative solution was right in front of me: I resorted to web scraping to get myself a nice table that I could quickly convert into a dataframe.

This is the code that let me scape the data:

```python
import requests
from bs4 import BeautifulSoup
city = input("Please enter a city: ")
r = requests.get('http://worldpopulationreview.com/us-cities/' + city + '-population/')
c = r.content
soup = BeautifulSoup(c, "lxml")
soup
main_content = soup.find('div', attrs = {'class': 'section-container clearfix'})
main_content
rows = main_content.find_all('tr')
```

As you can see above, the code is general and can be applied to any city in the US, depending on what the user enters in the input dialog box. Before I scaffolded each cell to make it completely general, I was using Portland, OR as my city of interest. I wanted to view its population growth over time, since it is a city that has recently popped up as a great place to live.

After I scraped the data, I did a fair amount of data cleaning because the format of the data from HTML inspection was not how I wanted it. I used the `re` library to accomplish string manipulation tasks.

The following two lines effectively initialized a writer which would be instrumental in making our animation object using the `matplotlib.animation.FuncAnimation` function:

```python
Writer = animation.writers['ffmpeg']
writer = Writer(fps=len(overdose[title])/2, metadata=dict(artist='Me'), bitrate=1800)
```

After that, I added a few lines of code to set up the figure and label my axes. Then, it was time to create the animation:

```python
def animate(j):
    data = overdose.iloc[:int(j+1)] #select data range
    p = sns.lineplot(x=data.index, y=data[title], data=data, color="b")
    p.tick_params(labelsize=14)
    plt.setp(p.lines,linewidth=2)
    
ani = animation.FuncAnimation(fig, animate, frames=len(overdose[title]), repeat=True)
ani.save(city + 'PopGrowth.gif', writer='imagemagick', fps=50)
```

The `frames` keyword argument in the `animation.FuncAnimation` call is probably the most critical part of the above code snippet. The number of frames determines how often `animate(i)` is going to be called, so in my case, it must correspond to the number of data points to be plotted. Then, in order to actually view the animation object `ani`, it must be saved as a .mp4 or .gif file.

This is how it turned out:
![portland pop growth](portlandPopGrowth.gif)
![seattle pop growth](seattlePopGrowth.gif)
![austin pop growth](AustinPopGrowth.gif)
![boston pop growth](BostonPopGrowth.gif)

EXTENSION FEATURE: Comparisions (Multiple Animations, One Figure)
------

I still wasn't satisfied after doing some single-line animations. I thought to myself, _how cool would it be if I could watch multiple snake-looking lines traversing through the figure like I'm watching a race happen before my eyes, except with data?_. So I decided to go make a new jupyter notebook and see if I could scrape data from multiple websites, make a dataframe with multiple y columns, and then attempt to plot multiple `sns.lineplots` in the same figure.

This is the function that allowed me to accomplish this:

```python
def make_df(i):
    # data extraction and data cleaning
    r = requests.get('http://worldpopulationreview.com/us-cities/' + cities[i] + '-population/')
    c = r.content
    soup = BeautifulSoup(c, "lxml")
    soup
    main_content = soup.find('div', attrs = {'class': 'section-container clearfix'})
    main_content
    rows = main_content.find_all('tr')
    year = []
    pop = []
    rows = rows[1:]
    for row in rows:
        row_td = row.find_all('td')
        str_cells = str(row_td)
        cleantext = BeautifulSoup(str_cells, "lxml").get_text()
        cleantext = cleantext[1:-1]
        text = cleantext.split(', ')
        year.append(int(text[0]))
        pop.append(int(text[1].replace(',', '')))

    #make dataframe
    cities[i] = (cities[i])[0].upper() + (cities[i])[1::]
    x = np.array(year)
    y = np.array(pop)
    XN,YN = augment(x,y,10)
    data = {'Population': YN, 'Year': XN}
    augmented = pd.DataFrame(data, columns = ['Year', 'Population'])
    df1 = augmented
    df1 = df1[::-1]
    return df1
```

I called this function 3 times to make 3 separate dataframes.

```python
# make a separate dataframe for each city
dfs = []
for i in range(3):
    dfs.append(make_df(i))
```

Then, I learned a VERY handy trick: merging dataframes!

```python
from functools import reduce
df_final = reduce(lambda left,right: pd.merge(left,right,on='Year'), dfs)
```

This is what the merged dataframe looks like:

![dataframe](dataframe.png)

And this is what the plot looks like once it has finished with the animation:

![comparison](comparison.png)

The actual animation, if you made it this far (thank you for reading if you did!)...

![multiple lines, one fig](test.gif)

