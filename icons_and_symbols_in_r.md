# Icons and symbols in R

Chen Jin




```r
library(readr)
library(ggpubr)
library(showtext)
library(plyr)
library(ggplot2)
library(ggimage)
library(emojifont)
library(personograph)
```

## Why use Symbols?

In class, we've been covering a variety of graph types, fundamental plotting techniques, and the proper treatments of different data we get. Equipped with these skills, we'd be able to come up with quite neat and informative visualizations. However, simply taking care of the most basic requirements would only generate an ubiquitous standard-following graph. These graphs may be suitable for a scientific research paper or a presentation on a formal meeting, but there are cases when we need a more lighthearted graph to arouse the viewer's interest. For instance, in an internet blog or a tweet, we would like the graph to be more entertaining and eye-catching to achieve more views. And one way of doing that is to add easily understandable symbols to the graph.

## 'pch' in Base R

For those of us who've had experience with base R for plotting before, we are probably quite familiar with the `pch` parameter. `pch` stands for 'plotting character', namely symbol to use. It is the standard argument to set the character that will be plotted in a number of R functions. 'pch' can either be a single character or an integer code for one of the set of graphic symbols. 

By using the library `{ggpubr}`, we can see the corresponding `pch` characters from 1 to 25 as below:

```r
# x <- c(sapply(seq(5, 25, 5), function(i) rep(i, 5)))
# y <- rep(seq(25, 5, -5), 5)
# 
# plot(x, y, pch = 1, cex = 3, yaxt = "n", xaxt = "n",
#      ann = FALSE, xlim = c(1, 29), ylim = c(0, 30), lwd = 1)
# text(x - 1.5, y, 1:25)

ggpubr::show_point_shapes()
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-3-1.png" width="672" style="display: block; margin: auto;" />

#### ``pch`` values
Values of `pch` are stored internally as integers. The interpretation is

* __NA_integer_ __: no symbol.
* __0 : 18__: S-compatible vector symbols.
* __19 : 25__: further R vector symbols.
* __26 : 31__: unused (and ignored).
* __32 : 127__: ASCII characters.
* __128 : 255__: native characters only in a single-byte locale and for the symbol font. (128:159 are only used on Windows.)
* __-32 ... __: Unicode code point (where supported).

#### An example of using `pch` (Iris Data)
By using different symbols for different species, we can clearly see the variations among the three types.

```r
# Define color for each of the 3 iris species
colors <- c("#00AFBB", "#E7B800", "#FC4E07")
colors <- colors[as.numeric(iris$Species)]

# Define shapes
shapes = c(16, 17, 18) 
shapes <- shapes[as.numeric(iris$Species)]

# Plot
plot(x = iris$Sepal.Length, y = iris$Sepal.Width,
     xlab = "Sepal Length", ylab = "Sepal Width", main = "Sepal Width vs. Length for 3 Species of Iris",
     col = colors, pch = shapes)
legend("topright", legend = levels(iris$Species),
      col =  c("#00AFBB", "#E7B800", "#FC4E07"),
      pch = c(16, 17, 18) )
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-4-1.png" width="672" style="display: block; margin: auto;" />

#### Limitations of `pch`:

As we can see from the list, the supported symbols are quite limited in R (basically from pch = 1-25 and some ASCII symbols). As the pch numbers goes up, you might encounter the issue that your locale does not support a particular character. Besides, it's very difficult to try to memorize what each number represents in this cumbersome list, so we would have to always look it up each time we need a particular symbol and check for its availability. Therefore, we will introduce some other appealing techniques as follows.


## showtext
`Showtext` is a package originally designed to deal with font issues in R. In many cases, using non-standard fonts in R graphs is not an easy task. For instance, using chinese characters in plotting would oftentimes result in unreadable results. 

Here's a trick that we can apply to build icons with the help of `showtext`. There are many fonts that look in the way of symbols, the `'wmpeople1'` demonstrated below is just one of them. Some other examples of symbol fonts include `Wingdings`, `Dingbat`, `webdings` and etc. By adding these fonts, we then manage to show the symbols as text in a graph. And now the `geom_text()` is something that we are all familiar with and we can easily handle and proceed.

#### An example of `showtext` (Demographics Data)
This example shows demographics by splitting the population into each subcategories of the education level. The use of symbol clearly indicates it's a demographic plot by nature and we can clearly distinguish between men and women with their corresponding token.

```r
font_add("wmpeople1", "resources/icons_and_symbols_in_r/wmpeople1.TTF")
showtext_auto()

dat = read.csv(textConnection('
      edu,educode,gender,population
      No School,1,m,17464
      No School,1,f,41268
      Primary School,2,m,139378
      Primary School,2,f,154854
      Middle School,3,m,236369
      Middle School,3,f,205537
      High School,4,m,94528
      High School,4,f,70521
      Bacherlor or above,5,m,57013
      Bacherlor or above,5,f,50334
'))

dat$int = round(dat$population / 10000)
gdat = ddply(dat, "educode", function(d) {
    male = d$int[d$gender == "m"]
    female = d$int[d$gender == "f"]
    data.frame(gender = c(rep("m", male), rep("f", female)),
               x = 1:(male + female))
})

gdat$char = ifelse(gdat$gender == "m", "p", "u")
ggplot(gdat, aes(x = x, y = factor(educode))) +
    geom_text(aes(label = char, colour = gender),
              family = "wmpeople1", size = 8) +
    scale_x_continuous("Population（10 million）") +
    scale_y_discrete("Education Level",
        labels = unique(dat$edu[order(dat$educode)])) +
    scale_colour_hue(guide = FALSE) +
    ggtitle("2012 Demographics Data")
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-5-1.png" width="672" style="display: block; margin: auto;" />

Every symbol in the above graph is actually a character of `p` or `u`. It's just they appear differently under this `'wmpeople1'` font. By applying this concept, we now have access to a great number of text symbols by simply adding font resources to our code.

## ggimage
`ggimage` supports image files and graphic objects to be visualized in the `ggplot2` graphic system. It includes a variety of interesting and useful functions and we will introduce some of them here.

### geom_image()
To have access to a greater range of symbol choices, we would sometimes want to use image from external sources as an icon in plotting. The `geom_image()` layer in ggplot2 can amazingly achieve to do that. We can simply use the url or file directory to import an image into our graphs.

For example, this is a plot that lines up several Rstudio icons in an "R" shape:

```r
x <- c(2, 2, 2, 2, 2, 3, 3, 3.5, 3.5, 4)
y <- c(2, 3, 4, 5, 6, 4, 6, 3, 5, 2)
d <- data.frame(x = x, y = y)

img <- ("https://www.r-project.org/logo/Rlogo.png")
ggplot(d, aes(x, y)) + geom_image(image = img, size = .1) +
  xlim(0, 6) + ylim(0, 7) +
  ggtitle("Plot R with Rstudio Symbols")
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-6-1.png" width="672" style="display: block; margin: auto;" />

### geom_emoji()
As we discovered previously, the symbols `pch` provides are mostly simple shapes or plain texts of ASCII characters. It doesn't offer much support if we want to use colorful emojis. This special layer of `geom_emoji()` would facilitate an easy incoporation of emojis into our graphs, by simply specifying their unicode.

#### Emoji Characters
Each emoji has a corresponding unicode. We can take a look at some examples of the Emoji unicode here https://apps.timwhitlock.info/emoji/tables/unicode. 

The `search_emoji()` function can also help us to find the related emojis that we are looking for. It will return emoji aliases which can be converted to unicode by emoji function.

```r
search_emoji('smile')
```

```
## [1] "smiley"      "smile"       "sweat_smile" "smiley_cat"  "smile_cat"
```

```r
emoji(search_emoji('smile'))
```

```
## [1] "😃" "😄" "😅" "😺" "😸"
```

#### An example of `geom_emoji()` (Iris Data)
We can visualize and distinguish the points that are fitted well by linear regression and those that are fitted relatively poorly with the help of intuitive emojis.

```r
set.seed(0)
iris2 <- iris[sample(1:nrow(iris), 30), ]
model <- lm(Petal.Length ~ Sepal.Length, data = iris2)
iris2$fitted <- predict(model)

p <- ggplot(iris2, aes(x = Sepal.Length, y = Petal.Length)) +
  geom_linerange(aes(ymin = fitted, ymax = Petal.Length),
                 colour = "red") +
  geom_abline(intercept = model$coefficients[1],
              slope = model$coefficients[2]) +
  ggtitle("Regression on Petal Length and Sepal Length")

p + ggimage::geom_emoji(aes(image = ifelse(abs(Petal.Length-fitted) > 0.5, '1f645', '1f600')), cex=0.04)
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-8-1.png" width="672" style="display: block; margin: auto;" />


### geom_icon()

`geom_icon()` provides support for icons on https://ionic.io/ionicons. This is a website that provides open source icons for web use. Figures are in black and white but are very well designed. It has each icon in three formats - 'Outline', 'Filled', and 'Sharp'. We can browse using key words to search for the icons conveniently.

#### An example of `geom_icon()`
Here's an example that I can list my daily routines using the wide variety of symbols from the ionic website.

```r
img <- list.files(system.file("extdata", package="ggimage"),
                  pattern="png", full.names=TRUE)
d <- data.frame(x = rep(1:5, 3),
                y = (rep(3:1, each = 5)))
d$icon <- c('bed', 'fast-food', 'bus', 'business', 'book', 'call', 'ice-cream', 'mail-unread', 'musical-notes', 'flask', 'language', 'pizza', 'beer', 'walk' ,'bed')
ggplot(d, aes(x,y)) + geom_icon(aes(image=icon)) + 
  xlim(0, 10) + ylim(0, 4) + 
  geom_text(aes(label = icon), size=2, vjust = -4) + 
  ggtitle("Daily Tasks in Icons") +
  theme_void()
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-9-1.png" width="672" style="display: block; margin: auto;" />

### geom_pokemon()
This is a special layer that provides support for pokemon characters. We can easily create cute graphs that target kids or anime fans. There is also a layer `geom_flags` that matches abbreviated country code to its corresponding national flags. That one can be particularly helpful in plotting worldwide data and making comparisons among different nations.

#### An example of `geom_pokemon()`
Let's find out where is pikachu located in this graph. How's its Attack and Defense ability compare to other pokemons shown in this selected group?

```r
pokemon <- readr::read_csv("https://gist.githubusercontent.com/armgilles/194bcff35001e7eb53a2a8b441e8b2c6/raw/92200bc0a673d5ce2110aaad4544ed6c4010f687/pokemon.csv")
pkm <- pokemon[21:51, ]
ggplot(pkm, aes(Attack, Defense)) + 
  geom_pokemon(aes(image=tolower(Name)), size=.05) + 
  geom_text(aes(label = Name), size=1.2, vjust = -3) + 
  ggtitle("Defense vs. Attack for Selected Pokemon")
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-10-1.png" width="672" style="display: block; margin: auto;" />


## Special Type: Personographs
A personograph (Kuiper-Marshall plot) is a pictographic representation of relative harm and benefit from an intervention. Each icon on the grid is colored to indicate whether that percentage of people is harmed by the intervention, would benefit from the intervention, has good outcome regardless of intervention, or bad outcome regardless of intervention. 

#### Arguments of `personograph` : 
* __data__: A list of names to percentages (from 0 to 1)

* __fig.title__: Figure title

* __draw.legend__: Logical if TRUE (default) will draw the legend

* __icon__: A grImport Picture for the icon, overwrites icon.style

* __icon.dim__: The dimensions of icon as a vector c(width, height) as numerical. Calculated from the dimensions if not supplied

* __icon.style__: A numeric from 1-11 indicating which of the included icons to use, they are mostly variations on the theme

#### Adjust grid size and Color each portions
We can specify the number of icons and the dimensions for the personograph. The default would be a 10 by 10 grid.

```r
data <- list(first=0.89, second=0.06, third=0.05)
# With colors
personograph(data, n.icons=64, dimensions=c(8, 8), colors=list(first="grey", second="blue", third="red"))
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-11-1.png" width="672" style="display: block; margin: auto;" />

#### Different icon style
We can change the icons by choosing from the preset icon styles or use grImport picture to overwrite it ourselves.

```r
# With different icon.style
data <- list(first=0.2, second=0.7, third=0.1)
personograph(data, n.icons=9, icon.style=6) # numeric from 1-11
```

<img src="icons_and_symbols_in_r_files/figure-html/unnamed-chunk-12-1.png" width="672" style="display: block; margin: auto;" />

This type of graph is particularly relevant and useful if we want to showcase the impact of some pandemic data. It can vividly depicts the transmission pattern and how the virus is influential to groups of people. The `personograph()` function is implemented in such a way that it's easy to just pass a named list of percentages, colors, and an icon. Since we can also overwrite the symbol, we can easily adapt the personograph to be suitable for other sanarios as well.


## References:

* R Documentation

* https://cran.r-project.org/web/packages/showtext/index.html

* https://github.com/GuangchuangYu/ggimage

* https://ionic.io/ionicons

* https://apps.timwhitlock.info/emoji/tables/unicode

* https://warwick.ac.uk/fac/sci/wdsi/events/wrug/resources/emoji_plots.pdf

* https://cran.r-project.org/web/packages/personograph/README.html








