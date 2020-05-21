

In order for me to get this to successfully compile, I needed to make a number of changes.

These errors likely could have been avoided other ways, but for the moment ...


```
Chromosome.cpp: In constructor 'Chromosome::Chromosome(const string&, unsigned int, bool)':
Chromosome.cpp:42:15: error: '*<unknown>.Chromosome::gs' is used uninitialized in this function [-Werror=uninitialized]
   42 |     if (this->gs) delete gs;
      |         ~~~~~~^~
```

so I commented out line 42 in src/libStatGen/general/Chromosome.cpp




```

Parameters.cpp: In member function 'virtual void LongParameters::Status()':
Parameters.cpp:573:25: error: use of an operand of type 'bool' in 'operator++' is deprecated [-Werror=deprecated]
  573 |             legacy_count++;
      |                         ^~
```

so I changed type from bool to int on line 556 of libStatGen/general/Parameters.cpp



```
StringBasics.cpp: In static member function 'static void String::check_vsnprintf()':
StringBasics.cpp:1398:40: error: '%5s' directive output truncated writing 9 bytes into a region of size 5 [-Werror=format-truncation=]
 1398 |         int check = snprintf(temp, 5, "%5s", "VSNPRINTF");
      |                                        ^~~   ~~~~~~~~~~~
In file included from /usr/include/stdio.h:867,
                 from StringBasics.h:21,
                 from StringBasics.cpp:18:
```

I don't like this one. It is checking something to test that it is truncated which raises a truncation error.

My quick solution to compilation is simply setting a variable outside the conditional block.

I inserted 
```
vsnprintfChecked = VSNPRINTF_IS_OK;
```
at line 1393 of src/libStatGen/general/StringBasics.cpp

Again, I'd prefer a better way.






I anticipated having to change isnan to std::isnan in src/invNorm/Main.cpp but that has been done already.




Now in a my docker file, `docker/ImputationPrep.Dockerfile` ...

```
FROM ubuntu:latest
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get -y update ; apt-get -y full-upgrade ; \ 
  apt-get -y install git python software-properties-common default-jdk wget curl htop make gcc zlib1g-dev libncurses5-dev g++ vim cmake ; \
  apt-get -y autoremove
# apt-get -y install git python3 python3-pip r-base software-properties-common default-jdk wget ; \

# python-pip for python2 hard to find
RUN curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"; python get-pip.py

RUN git clone https://github.com/ucsffrancislab/gotcloud.git ; cd gotcloud/src ; make
```

This works

```
docker build -t impute --file docker/ImputationPrep.Dockerfile ./
docker run --rm -it impute
```


This is all rather excessive since all I wanted was vcfCooker. And there are probably other utilities out there that do the same thing!

