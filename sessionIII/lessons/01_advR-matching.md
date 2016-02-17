---
title: "Data manipulation"
authors: Meeta Mistry and Mary Piper
date: "01/21/16"
layout: topic
minutes: 90
---

## Learning Objectives
* Using nested functions to improve efficiency in executing commands
* Learning how to match and re-order data 

## Nested functions

Thus far, to perform any specific task, we have executed every function separately; if we wanted to use the results of a function for downstream purposes, we saved the results to a variable. As you become more comfortable with R, you will find that it is more efficient to code using nested functions, or functions within other functions, which will allow you to execute multiple commands at the same time.

Even if you decide to avoid writing nested functions for the time being, you should still have experience reading and understanding them. The key to understanding nested functions is to **read from the inside out**.

### Nested functions practice #1

You realize that you forgot to include important metadata regarding sex of your samples in your `metadata` file. You would like to add this data to your `metadata` dataframe, and using functions separately would require us to execute three separate steps:  

**Step 1:** Create the `sex` vector: 
	
	sex <- c("M","F","M","M","F","M","M","F","M","M","F","M")
	
**Step 2:** Turn the `sex` vector into a factor variable:
	 
	 sex_fr <- factor(sex)
	 
**Step 3:** Use the `cbind` function to add the column to the **end** of the `metadata` dataframe: 

	metadata <- cbind(metadata, sex=sex_fr)

Instead of performing all three steps, we would like to create a nested function. **To create a nested function, simply replace the variable name with its contents**. We could combine steps 1 and 2 by replacing `sex` in **Step 2** with it's contents (`c("M","F","M","M","F","M","M","F","M","M","F","M")`):

	sex_fr <- factor(c("M","F","M","M","F","M","M","F","M","M","F","M"))
	metadata <- cbind(metadata, sex=sex_fr)
	
It is possible to combine all steps, but your code would be difficult to read, so we don't recommend doing this:

	metadata <- cbind(metadata,
			sex=factor(c("M","F","M","M","F","M","M","F","M","M","F","M")))

### Nested functions practice #2			
Now, let's say that you are interested in counting the number samples in your dataset that have  "Wt" genotype within our `metadata` file. Obviously, for our small file, we could just look at the file, but if we had too many samples to count, we could do the following:

**Step 1:** Determine the **location** of samples with `genotype` equal to "Wt":
	
	wt_loc <- which(metadata$genotype == "Wt")
	
**Step 2:** Extract the samples with "Wt" `genotype`:
	
	wt <- metadata$genotype[wt_loc]

**Step 3:** Determine the number of samples with `genotype` "Wt":
	
	length(wt)
	
Alternatively, we could combine all steps:

	length(metadata$genotype[which(metadata$genotype == "Wt")])

Learning to understand nested functions is a critical part of your mastery of R. Not only will their use improve your efficiency, but nested functions are frequently encountered in help forums and R package documentation, so understanding them is critical to your learning process. 

## Matching data 

Often when working with genomic data, we have a data file that corresponds with our metadata file. The data file contains measurements from the biological assay for each individual sample. In our case, the biological assay is gene expression and data was generated using RNA-Seq. Let's first download a file of RPKM values into our `data/` folder. To do so use the "Save link as.." after right clicking on [this link](https://github.com/hbc/NGS_Data_Analysis_Course/blob/master/sessionII/data/counts.rpkm)

	rpkm_data <- read.csv("data/counts.rpkm")

Take a look at the first few lines of the data matrix to see what's in there.


	head(rpkm_data)


It looks as if the sample names (header) in our data matrix are similar to the row names of our metadata file, but it's hard to tell since they are not in the same order. We can do a quick check of the dimensions and at least see if the numbers match up. 


	ncol(rpkm_data)


What we want to know is, **do we have data for every sample that we have metadata?** 

## The `%in%` operator
 
Although lacking in [documentation](http://dr-k-lo.blogspot.com/2013/11/in-in-r-underused-in-operator.html) this operator is well-used and convenient once you get the hang of it. The operator is used with the following syntax: 

	vector1_of_values %in% vector2_of_values


It will take a vector as input to the left and will **evaluate each element to see if there is a match in the vector that follows on the right of the operator.** *The two vectors do not have to be the same size.* This operation will return a vector of the same length as vector1 containing logical values to indicate whether or not there was a match. Take a look at the example below:


	A <- c(1,3,5,7,9,11)   # odd numbers
	B <- c(2,4,6,8,10,12)  # even numbers

	# test to see if any of A are in B	
	A %in% B


```
## [1] FALSE FALSE FALSE FALSE FALSE FALSE
```

Since vector A contains only odd numbers and vector B contains only even numbers, there is no overlap and so the vector returned contains a `FALSE` for each element. Let's change a couple of numbers inside vector B to match vector A.


```r
B <- c(2,4,6,8,1,5)  # add some odd numbers in 

# test to see if any of A are in B
A %in% B
```

```
## [1]  TRUE FALSE  TRUE FALSE FALSE FALSE
```

The logical vector returned tells us which elements are matching and which are not.  In this example the vectors are small and so it's easy to count by eye; but when we work with large datasets this is not practical. A quick way to assess whether or not we had any matches would be to use the `any` function to see if **any of the values contained in vector A are also in vector B**:

	any(A %in% B)


The `all` function is also useful. Given a logical vector, it will tell you whether are all values returned are `TRUE`. If there is at least one `FALSE` value, the `all` function will return a `FALSE` and you know that all of A are not contained in B.


	all(A %in% B)

Suppose we had **two vectors that had the same values but just not in the same order**. We could also use `all` to test for that. Rather than using the `%in%` operator we would use `==` and compare each element to the same position in the other vector. Unlike the `%in%` operator, **for this to work you must have two vectors that are of equal length**.

```r
A <- c(1,2,3,4,5)
B <- c(5,4,3,2,1)  # same numbers but backwards 

# test to see if any of A are in B
A %in% B

# test to see if any of A is equal to B
A == B

# use all to check if they are a perfect match
all(A == B)

```

Let's try this on our data and see whether we have metadata information for all samples in our expression data. We'll start by creating two vectors; one with the `rownames` of the metadata and `colnames` of the RPKM data. These are base functions in R which allow you to extract the row and column names as a vector:

	x <- rownames(metadata)
	y <- colnames(rpkm_data)

Now check to see that all of `x` are in `y`:

	all(x %in% y)

*Note that we can use nested functions in place of `x` and `y`:*

	all(rownames(metadata) %in% colnames(rpkm_data))
	
We know that all samples are present, but are they in the same order:

	all(rownames(metadata) == colnames(rpkm_data))

**Looks like all of the samples are there, but will need to reordered.**

***
**Exercise** 

We have a list of IDs for marker genes of particular interest. We want to extract count information associated with each of these genes, without having to scroll through our matrix of count data. We can do this using the `%in%` operator to extract the information for those genes from `rpkm_data`.

1. Create a vector for your important gene IDs, and use the %in% operator to determine whether these genes are contained in the row names of our `rpkm_data` dataset.

		important_genes <- c("ENSMUSG00000083700", "ENSMUSG00000080990", "ENSMUSG00000065619", "ENSMUSG00000047945", "ENSMUSG00000081010", 	"ENSMUSG00000030970")
	
2. Extract the rows containing the important genes from your `rpkm_data` dataset.	

***

### The `match` function

We'll be using the `match` function to evaluate which samples are present in both dataframes, and then re-order them. This function takes at least 2 arguments: 

1. a vector of values to *be matched*
2. a vector of values to be *matched against*. 

The function returns the position in the second vector of the matches. Let's create vectors `first` and `second` to demonstrate how it works:

	first <- c("A","B","C","D","E")
	second <- c("E","D","C","B","A")  # same letters but backwards 
	
	match(first,second)
	
	[1] 5 4 3 2 1

The function should return a vector of size `length(first)`. Each number that is returned represents the index of the `second` vector where the matching value was observed. 

Let's change vector `second` so that only a subset are retained:

	first <- c("A","B","C","D","E")
	second <- c("D","B","A")  # remove values 

And try to `match` again:

	match(first,second)

	[1]  3  2 NA  1 NA

Note, for values that don't match by default return an `NA` value. You can specify what values you would have it assigned using `nomatch` argument. Also, if there is more than one matching value found only the first is reported.

### Reordering data using indexes
Indexing `[ ]` can be used to extract values from a dataset as we saw earlier, but we can also use it to rearrange our data values. For example, we can reorder the `second` vector using the indexes from the `match` function of where the elements of the `first` vector occur in the `second` vector.

First, we save the match indexes to a variable:

	first <- c("A","B","C","D","E")
	second <- c("E","D","C","B","A")  
	
	idx <- match(first,second)
	
	idx
	
	[1] 5 4 3 2 1

Then, we can use the indexes to reorder the elements of the `second` vector to be in the same positions as the matching elements in the `first` vector:

	second_reordered <- second[idx]

	second_reordered
	
### Reordering genomic data using `match` function
Using the `match` function, we now would like to *match the column names of our metadata to the row names of our expression data*, so these will be the arguments for `match`. Using these two arguments we will retrieve a vector of match indexes. The resulting vector represents the re-ordering of the column names in our data matrix to be identical to the rows in metadata:
 
	rownames(metadata)
	
	colnames(rpkm_data)
	
	idx <- match(rownames(metadata), colnames(rpkm_data))


Now we can create a new data matrix in which columns are re-ordered based on the match indices:

	rpkm_ordered  <- rpkm_data[,idx]


Check and see what happened by using `head`. You can also verify that column names of this new data matrix matches the metadata row names by using the `all` function:

	head(rpkm_ordered)
	all(row.names(metadata) == colnames(rpkm_ordered))

***
**Exercise** 


Your collaborators sent you a new dataset containing additional metadata information for all of the samples in your current metadata file. You would like to combine the two files. To do so, perform the following steps: 

1. Read in the `metadata2` tab-delimited file and save it as a dataframe called `metadata2`.
2. Check to make sure you have the same samples in both datasets.
3. To ensure you are adding the correct metadata to each sample, you need to make sure that the samples are in the same order before you combine them. Reorder one of the datasets using the `match()` function. **Try to use nested functions.**
4. Combine the two datasets, and save the large dataset as `metadata_combined`.
4. Write the `metadata_combined` to file using `write.table()`, and save as a tab-delimited file called `metadata_combined.txt`. 

---
*The materials used in this lesson is adapted from work that is Copyright © Data Carpentry (http://datacarpentry.org/). 
All Data Carpentry instructional material is made available under the [Creative Commons Attribution license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0).*
