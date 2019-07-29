---
layout: post
title:  "Extracting Data from Text with Python"
author: Shawn Hermans
date:   2016-02-29 10:00:00 -0600
categories: [tutorial, python, text, data]  
---


## Working with text data

If you have conducted any type of data analysis, you probably have worked with typical numeric and text data types.  Microsoft Excel supports various numeric and text types.  If you work with databases, you have probably used decimals, floating point numbers, integers, and character types.  

While the standard data types are good enough for most applications, sometimes you have text data with some sort of structure. Social security numbers, mailing addresses, phone numbers, email addresses,  and website URLs are some typical examples.  In these cases, treating the data as unstructured text might not be good enough.  If you are examining data with telephone numbers, you might want to compare records within the same area code.  If the area code is not stored in its own field, you will need to extract from the full phone number.  Maybe you want to look at records by zipcode, but all you have is a mailing address.

At Bellevue University, there are various examples of useful information being embedded in text data.  Course names follow a naming convention that contains the program code, course number, section number, and term information.   In the Skills to Performance model, information is embedded in the naming convention for the skills and rubrics.   Extracting information from this type of data is exceedingly useful to analytics efforts.

In addition to extracting information from text fields, it is often necessary to validate that a text field conforms to a particular naming convention.  If data does not conform to the naming convention, it might prevent analysis of that data.  At the very least, data that does not conform to the naming convention causes extra work for everyone involved and should be avoided when possible.

This document provides a short introduction on how to use the Python programming language to work with semi-structured text.  It uses examples from Bellevue University's Skills to Performance naming convention.  This tutorial demonstrates how to use Python to determine whether a skill name adheres to the naming convention.  Furthermore, it shows how to extract information embedded in the skill name.

## Skill naming conventions

The Skills to Performance model evaluates student performance based upon mastery of skills.   Courses within the same program may share common skill definitions.  Conversely, a skill definition might only apply to a single course.  This is often the case with general education courses that do not belong to a particular program.

This tutorial will look at two different skill identifier formats.  One format is used for general education courses and the other is used for skills attached to a program.  

### General education skill name

General education skills uses the following naming conventions:

1.  `<course_abbr>_<course_num>_<gen_ed_cat>.<skill>.<subskill>`
2.  `<course_abbr>_<course_num>_<gen_ed_cat>.<skill>`

`HI_150_HC.01.02` is an example of skill definition for a general education course.  This example uses the first format. `HI_150_HC.01` is a case without a subskill. It uses the second form.

* **course_abbr** is the abbreviated name for the course.   This abbreviation should be no shorter than two characters and no longer than five characters.  The characters should always be upper case letters.
* **course_num** is the three digit number for the course.
* **gen_ed_cat** is the general education category for the course. It is two upper case letters.
* **skill** is the two digit number identifying the skill.  It must contain a leading zero if the number is less than 10.
* **subskill** is the two digit number identifying the subskill.  It must include a leading zero if the number is less than 10.  Subskill is optional if the parent skill does not define any subskills. If no subskills are defined, the second form should be used.

### Program skill name

Skills attached to a program have the following naming conventions:

1.  `<program_abbr>_<course_num>_<perf_obj>.<skill>.<subskill>`
2.  `<program_abbr>_<course_num>_<perf_obj>.<skill>`

`PPSY_300_01.01.02` is an example of skill definition for a course belonging to a program.  This example uses the first format. `PPSY_300_01.01` is a case without a subskill. It uses the second form.

* **program_abbr** is the abbreviated name for the program.   This abbreviation should be no shorter than two characters and no longer than five characters.  The characters should always be upper case letters.
* **course_num** is the three digit number for the course.
* **perf_obj** is the performance objective number for the skill. It is always two digits and must contain a leading zero if the number is less than 10.
* **skill** is the two digit number identifying the skill.  It must contain a leading zero if the number is less than 10.
* **subskill** is the two digit number identifying the subskill.  It must include a leading zero if the number is less than 10.  Subskill is optional if the parent skill does not define any subskills. If no subskills are defined, the second form should be used.

## Validating and parsing skill names

The code presented in this tutorial has been simplified to help those who are not familiar with Python.  It avoids advanced Python features and is heavily commented.  It only uses features found the Python 3 standard library.

If you want to run the examples given in this document, you will need to install Python 3 on your system.  As of this writing, Python 3.5 is the latest stable version of Python 3.  If you are new to Python, I recommend using the [Anaconda distribution](https://www.continuum.io/downloads) of Python produced by Continuum Analytics.  It provides graphical installers for both Windows and Mac OS X.  It also includes [Jupyter notebook](http://jupyter.org/); a locally run web application for creating documents powered by Python. In fact, I created this document with Jupyter notebook.

### Regular expressions

The code in this tutorial makes heavy use of regular expressions.  Regular expressions are usefully for matching patterns in text.  Consider a social security number as an example.  A social security number is three digits followed by a hyphen followed by two digits followed by another hyphen followed by four digits.  Explaining patterns like this in English is rather long and clumsy.

Defining this pattern as a regular expression is much more compact. The regular expression (regex for short) for a social security number is:

> `^([0-9]{3})-([0-9]{2})-([0-9]{4})$`   

Don't feel dejected if this looks like an incomprehensible jumble of numbers and symbols.  As conveyed in this [XCKD comic](http://imgs.xkcd.com/comics/perl_problems.png), even experienced developers struggle with regular expressions. While regular expressions can be frustrating at times, they are a powerful tool when working with partially structured data.

![Perl problems](http://imgs.xkcd.com/comics/perl_problems.png)

### Python code

The following section describes the skill parsing code.  If you want to try it out for yourself, this [skill_parser.py gist](https://gist.github.com/shawnhermans/72cf2dc1e37fc2615f05) provides the Python used to define the skill parsing code.  If you want the code, unit tests, and examples, it is all bundled in [this Jupyter notebook](https://gist.github.com/shawnhermans/e9452fec593a126322de).  Select the option to *Download ZIP*, unzip the contents, and open the contents using Jupyter notebook.  The end of this tutorial provides brief instructions on how to download and install Jupyter notebook.  
<script src="https://gist.github.com/shawnhermans/72cf2dc1e37fc2615f05.js">
</script>


### A few examples

The following shows how to use the code to validate and parse skill names.

#### General education skill without subskill


{% highlight python %}
gen_ed_skill = 'HI_150_HC.01'
parse_skill(gen_ed_skill)
{% endhighlight %}

Which returns the result:

    {'course': 'HI',
     'course_number': '150',
     'gen_ed_category': 'HC',
     'skill': '01',
     'subskill': None}



#### General education skill with subskill


{% highlight python %}
gen_ed_with_subskill = 'HI_150_HC.01.02'
parse_skill(gen_ed_with_subskill)
{% endhighlight %}

Which returns the result:


    {'course': 'HI',
     'course_number': '150',
     'gen_ed_category': 'HC',
     'skill': '01',
     'subskill': '02'}



#### Program skill without subskill


{% highlight python %}
program_skill = 'PPSY_300_01.01'
parse_skill(program_skill)
{% endhighlight %}

Which returns the result:


    {'course_number': '300',
     'performance_objective': '01',
     'program': 'PPSY',
     'skill': '01',
     'subskill': None}



#### Program skill with subskill


{% highlight python %}
program_with_subskill = 'PPSY_300_01.01.02'
parse_skill(program_with_subskill)
{% endhighlight %}

Which returns the result:


    {'course_number': '300',
     'performance_objective': '01',
     'program': 'PPSY',
     'skill': '01',
     'subskill': '02'}



#### Handling invalid input

When we try to parse an skill that does not meet the format, it will raise a ValidationError. This lets the developer know something went wrong.  


{% highlight python %}
try:
    # Python tries to execute the code in this block.  
    # If the code runs without error, print out parsed info.  
    # If the code raises a ValidationError, go to exception block

    invalid_skill = 'PPSY_300_01.1'
    print(parse_skill(invalid_skill))
except ValidationError as err:
    # Block catches ValidationError. Prints out error message
    print(err)
{% endhighlight %}

Which returns this result:

    'PPSY_300_01.1' does not match known format


### Unit testing

This section gives an overview of how to use unit testing in Python. Unit testing is a way of testing individual units of code.   When a developer submits code changes, a testing service automatically runs these tests.  The developer is not allowed to add the modifications to the project unless all the tests pass. The biggest advantage of unit testing (and automated testing in general) is it catches errors early in the development process.   

#### A simple test

The following is a single unit test.   It tests the parsing of a general education skill that does not contain any subskill.  When this test runs, it evaluates the assertion statements.  These statements are simple true/false statements.  If all the statements evaluate to true, the tests pass.  If an assertion fails (i.e. evaluates as false), the test fails.  The following unit test should run without any errors.


{% highlight python %}
def test_parse_gen_ed_skill():
    result = parse_skill('HI_150_HC.01')

    assert result['course'] == 'HI'
    assert result['course_number'] == '150'
    assert result['gen_ed_category'] == 'HC'
    assert result['skill'] == '01'
    assert result['subskill'] == None

try:
    test_parse_gen_ed_skill()
    print('Test passed!')
except AssertionError as err:
    print('Test failed :(')
    print(err)
{% endhighlight %}

Running this code should print:

    Test passed!


#### Failing the test

The next example shows what happens when a test fails. It is similar to the test above but has a failing assertion. Specifically, the skill string "PPSY_300_01.01" should return the skill number of "01".  In this particular example, we are telling it to match "1" and not "01".  Since these are not the same values, the test should fail.


{% highlight python %}
def test_parse_program_skill():
    result = parse_skill('PPSY_300_01.01')

    assert result['program'] == 'PPSY'
    assert result['course_number'] == '300'
    assert result['performance_objective'] == '01'

    # Define a message
    skill_error_msg = 'Received skill value '
    skill_error_msg += '"{0}". '.format(result['skill'])
    skill_error_msg += 'Expected "1" as value.'
    assert result['skill'] == '1', skill_error_msg

try:
    test_parse_program_skill()
    print('Test passed!')
except AssertionError as err:
    print('Test failed :(')
    print(err)
{% endhighlight %}

Running this code should result in a test failure.

    Test failed :(
    Received skill value "01". Expected "1" as value.


## Next steps

If you are interested in learning Python, there are plenty of resources available.  [Automate the boring stuff](https://automatetheboringstuff.com/) is a gentle introduction to Python that uses practical, real-world examples.  [Google's Python class](https://developers.google.com/edu/python/)  is a great resource if you already have some programming background.   [Data Science from Scratch](http://www.amazon.com/Data-Science-Scratch-Principles-Python/dp/149190142X/) and [Python for Data Analysis](http://www.amazon.com/Python-Data-Analysis-Wrangling-IPython/dp/1449319793/) are good books for those with a background in analysis.   

If you just want to download Python and try it out, I recommend starting with the Anaconda distribution from Continuum Analytics. They provide [graphical installers for Windows and Mac OS X](https://www.continuum.io/downloads).  Choose the Python 3.5 installer if you are not sure which version to pick. They also [provide installation instructions](http://docs.continuum.io/anaconda/install).

Once you have it installed, open up Jupyter notebook and create a Python 3 notebook.  You can use this notebook to experiment with Python code.
