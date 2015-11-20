# Creating a pull-request

Once you've discussed a feature, passed your tests and produced a working implementation, it's time to submit a pull-request for code review.

Code review takes place on GitHub in the relevant repository and usually takes a few days based on the importance and/or gravity of the change. It is also here where your code is inspected and discussed further. Maybe there is something not covered in the tests, maybe there is a more performance solution to a particular area or maybe the conventions need work.

The general idea is that once your change passes code review, it's on.

### Getting on

Once you've commited your changes to your GitHub fork, it's time to make the pull-request.

![image](https://cloud.githubusercontent.com/assets/2152766/11125115/0d4ec57c-8960-11e5-8e3a-1fa0419a898e.png)

Feel free to provide a meaningful description, raise any concerns or questions you might have about your work, otherwise it is assumed the code and commit messages speak for itself.

### Continuous integration

Once you're submitted a pull-request and automated testing session begins.

It will..

1. Run the same set of tests you yourself ran before submitting, including your new test, on multiple platforms and all supported versions of Python.
2. Perform a coverage check, to make sure the things you changed were properly tested.
3. Perform an overal inspection on code health. This include things such as following PEP08 conventions to whether functions are overly complicated.

Find out..

- [**More about coverage**](https://coverage.readthedocs.org/en/coverage-4.0.2/)
- [**More about automated testing on Linux**](http://travis-ci.org)
- [**More about automated testing on Windows**](http://www.appveyor.com)

![image](https://cloud.githubusercontent.com/assets/2152766/11125629/efab66d0-8962-11e5-8676-c51943656d1e.png)

There is nothing required on your part to actually perform these checks, but should any of them fail you are advised to try and resolve it.

Nothing generally gets merged without an all-clear.

Also note that you are welcome to submit pull-requests that you don't yet expect to pass these tests, in the interest of finding out what you have left to do. Each test will provide you with an amount of information useful in hunting down bugs or living up to the conventions and standards of the repository.

### The aftermath

From here, enjoy some peace and quiet and prepare for comments.

<div class="modified-date">{{ file.mtime }}</div>