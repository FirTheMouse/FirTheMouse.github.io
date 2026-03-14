I wanted to make a compiler, so I did.  
This is where I left it.  
Originally, this started as a one pass string-based parser, similar to my other early domain specific languages.  
I was making GDSL (lit. Golden Domain Specific Language) for GUIDE (Golden User Interface Development Environment) as a way to store the behaviour of dynamic gui elements with just a string.  
While writing the code, I had an idea to eliminate the syntactic overhead of the engine API  
i.e: GDSL was going to be a transpiler to C++, just a way to get rid of 'g\_ptr\<T\>' (my intrusive ref counting I use for memory management on my generic object).  
   g\_ptr\<Type\> Person \= make\<Type\>();  
   Person-\>add\_initializer(\[\](g\_ptr\<Object\> object){  
       object-\>set\<int\>("age",42);  
   }); //Explicit initializer addition

   (\*Person)+(\[\](g\_ptr\<Object\> object){  
       object-\>set\<int\>("age",42);  
   }); //Will be Person \+{ \* logic here \* }

   auto people \= make\<group\>(); //Will be: group people;  
   auto joe \= Person-\>create(); //Will be: Person joe;  
   \*people \<\< joe; //Will be: people \<\< joe;  
   auto man \= (\*people)\[0\]; //Will be: man people\[0\];  
   //print(man-\>get\<int\>("age")); //Will be: print(man.age);  
This was the original API sketch I had for how I wanted GDSL's syntax to look, at this point I had decided this would be a DSL for Golden (the engine) instead of just a script interpreter for GUIDE.  
By version 0.0.5 (which is archived in the giant storage dump in the main.cpp of testing) I had moved towards a multi-pass system which is more recognizable as GDSL's compiler, with the pipeline: token \-\> a\_node \-\> t\_node/s\_node. This was after about 4 days of development. At this time, I started working on the Type data structure as well. A big pain point with this version was adding types, which each required explicit handling such as int\_list, float\_list, and switch cases to detect them all.  
At this time, Type was based on the architecture of Scene, with the idea being we create a struct of arrays/ECS and use that as the memory model. Nowadays Scene’s pooling just uses Type, but Scene was the original, link to Type here for those curious about the code: [https://github.com/FirTheMouse/Golden/blob/main/Engine/include/core/type.hpp](https://github.com/FirTheMouse/Golden/blob/main/Engine/include/core/type.hpp)  
To make Type type agnostic, I came up with the idea to store the bytes directly, and bucket the arrays by size instead of by type. I eventually implemented this on day 6, and got it working in a very basic state with a clunky API, though in testing it matched a handmade struct of arrays. I actually made the classes now in Logger/rig (my profiler) such as time\_function for this.  
I used the new Type in version 0.0.6, and realized the next major pain point would be the switch cases. I didn't like the idea of having to go through a bunch of methods to add and maintain cases, it felt like something that would lead to points of failure and development pain later on. I also *really* didn't want to spend hours refactoring all the switch cases I had accumulated.  
I avoided this for 3 days, and eventually, I had the spike of motivation to just bite the bullet and get it done.  
It only took 5 hours to actually figure out how to make it modular with the registry and maps.  
After that, it was days of one to two hours of just sorting and converting, I would work on it while doing other things like during a long car drive or while watching a show. Eventually, I had everything arranged into the new module system.  
I did some more work to get GDSL to a functioning state, up to 0.1.0, then I had to go to college and deal with moving in and such.  
A couple weeks later, I did further work, all of which should be visible on GitHub now.  
At one point, while working on something else, I had the idea that maybe I could turn the GDSL compiler into a way to make  
DSLs quickly for various projects, instead of my quick string-based parsers I've made a couple times, yet I haven't tried this yet.

## **Demo Modules**

GitHub Link:  
[https://github.com/FirTheMouse/Golden/tree/main/Projects/GDSL](https://github.com/FirTheMouse/Golden/tree/main/Projects/GDSL)  
The code snippet I run with it is:

type person {  
    string name \= "noname";  
}

person joe;  
person mary;  
joe.name \= "Joe";  
string word;  
word \= joe.name;  
print("Word is: ",word,"\!");  
joe.name \= "Not Joe";  
print("The name of Joe is: ", joe.name);  
mary.name \= "Mary";  
print("The name of Mary is: ", mary.name);

The actual benchmarking code looks like this:

base\_module::initialize();  
variables\_module::initialize();  
literals\_module::initialize();  
opperator\_module::initialize();  
property\_module::initialize();  
control\_module::initialize();  
type\_module::initialize();  
data\_module::initialize();  
paren\_module::initialize();  
functions\_module::initialize();  
std::string code \= readFile("../Projects/GDSL/src/golden.gld");  
list\<g\_ptr\<Token\>\> tokens \= tokenize(code);  
list\<g\_ptr\<a\_node\>\> nodes \= parse\_tokens(tokens);  
balance\_precedence(nodes);  
g\_ptr\<s\_node\> root \= parse\_scope(nodes);  
parse\_nodes(root);  
discover\_symbols(root);  
g\_ptr\<Frame\> frame \= resolve\_symbols(root);  
//execute\_r\_nodes(frame);  
//Streaming  
// stream\_r\_nodes(frame);  
// execute\_stream(frame);

list\<list\<std::function\<void(int)\>\>\> f\_table;  
list\<list\<std::string\>\> s\_table;  
list\<vec4\> comps;  
int z \= 0;  
f\_table \<\< list\<std::function\<void(int)\>\>{};  
s\_table \<\< list\<std::string\>{};  
z \= 0;

s\_table\[z\] \<\< "execute\_r\_nodes"; //0  
f\_table\[z\] \<\< \[frame\](int i){  
execute\_r\_nodes(frame);  
};

s\_table\[z\] \<\< "stream\_r\_nodes"; //1  
f\_table\[z\] \<\< \[frame\](int i){  
if(frame-\>stored\_functions.length()==0) {  
stream\_r\_nodes(frame);  
}  
};  
s\_table\[z\] \<\< "execute\_stream"; //2  
f\_table\[z\] \<\< \[frame\](int i){  
execute\_stream(frame);  
};

s\_table\[z\] \<\< "CPP"; //3  
f\_table\[z\] \<\< \[\](int i){  
struct person {  
std::string name \= "noname";  
};  
person joe;  
person mary;  
joe.name \= "Joe";  
std::string word;  
word \= joe.name;  
print("Word is: ",word,"\!");  
joe.name \= "Is Joe";  
print("The name of Joe is: ", joe.name);  
mary.name \= "Mary";  
print("The name of Mary is: ", mary.name);  
};

comps \<\< vec4(0,0 , 0,3);  
comps \<\< vec4(0,2 , 0,3);  
run\_rig(f\_table,s\_table,comps,true,3,500);

The top part is just compilation, I commented out the normal execution so it runs through the test harness instead. This:  
Run\_rig(f\_table,s\_table,comps,true,1,500);   
Is providing the test cases, which to compare, enabling warmup runs, setting it to run each table once, and then running the tables 500 times to average it out.  This is so everything is being assessed equally, and cache warming should have minimal impact. Just in case, I arranged the run order of the cases so that the standard execution occurs first, as part of the streaming is standard execution, then streaming, then the C++ execution.   
Benchmarking results here:

### **Run 1**

Word is: Not Joe\!  
The name of Joe is: Not Joe  
The name of Mary is: Mary  
Word is: Joe\!  
The name of Joe is: Is Joe  
The name of Mary is: Mary  
Word is: Joe\!  
The name of Joe is: Is Joe  
The name of Mary is: Mary  
Word is: Joe\!  
The name of Joe is: Is Joe  
The name of Mary is: Mary  
\-------------------------  
       \==WARM==  
\-------------------------  
execute\_r\_nodes: 368066 ns (122689 ns per operation)  
stream\_r\_nodes: 77.542 ns (25.8473 ns per operation)  
execute\_stream: 56237.8 ns (18745.9 ns per operation)  
CPP: 34628 ns (11542.7 ns per operation)  
\-------------------------  
Factor \[execute\_r\_nodes/CPP\]: 10.6292 (execute\_r\_nodes is 962.916493% slower than CPP)  
Factor \[execute\_stream/CPP\]: 1.62406 (execute\_stream is 62.405744% slower than CPP)  
\-------------------------

I'm including the top print, which I'll disable after this, just to prove that it is in fact running correctly (the C++ prints "Is Joe" while GDSL prints "Not Joe").

### **Run 2**

\-------------------------  
      \==COLD==  
\-------------------------  
execute\_r\_nodes: 31583 ns (10527.7 ns per operation)  
stream\_r\_nodes: 15292 ns (5097.33 ns per operation)  
execute\_stream: 791 ns (263.667 ns per operation)  
CPP: 667 ns (222.333 ns per operation)  
\-------------------------  
Factor \[execute\_r\_nodes/CPP\]: 47.3508 (execute\_r\_nodes is 4635.082459% slower than CPP)  
Factor \[execute\_stream/CPP\]: 1.18591 (execute\_stream is 18.590705% slower than CPP)  
\-------------------------  
\-------------------------  
       \==WARM==  
\-------------------------  
execute\_r\_nodes: 23926.2 ns (7975.39 ns per operation)  
stream\_r\_nodes: 54.502 ns (18.1673 ns per operation)  
execute\_stream: 395.506 ns (131.835 ns per operation)  
CPP: 282.156 ns (94.052 ns per operation)  
\-------------------------  
Factor \[execute\_r\_nodes/CPP\]: 84.7976 (execute\_r\_nodes is 8379.763677% slower than CPP)  
Factor \[execute\_stream/CPP\]: 1.40173 (execute\_stream is 40.172812% slower than CPP)  
\-------------------------  
\==DONE==

### **Run 3**

\-------------------------  
      \==COLD==  
\-------------------------  
execute\_r\_nodes: 31875 ns (10625 ns per operation)  
stream\_r\_nodes: 14292 ns (4764 ns per operation)  
execute\_stream: 875 ns (291.667 ns per operation)  
CPP: 750 ns (250 ns per operation)  
\-------------------------  
Factor \[execute\_r\_nodes/CPP\]: 42.5 (execute\_r\_nodes is 4150.000000% slower than CPP)  
Factor \[execute\_stream/CPP\]: 1.16667 (execute\_stream is 16.666667% slower than CPP)  
\-------------------------  
\-------------------------  
       \==WARM==  
\-------------------------  
execute\_r\_nodes: 23330.9 ns (7776.98 ns per operation)  
stream\_r\_nodes: 53.762 ns (17.9207 ns per operation)  
execute\_stream: 402.324 ns (134.108 ns per operation)  
CPP: 267.59 ns (89.1967 ns per operation)  
\-------------------------  
Factor \[execute\_r\_nodes/CPP\]: 87.1891 (execute\_r\_nodes is 8618.908778% slower than CPP)  
Factor \[execute\_stream/CPP\]: 1.50351 (execute\_stream is 50.350910% slower than CPP)  
\-------------------------  
\==DONE==

### **Run 4**

\-------------------------  
      \==COLD==  
\-------------------------  
execute\_r\_nodes: 32666 ns (10888.7 ns per operation)  
stream\_r\_nodes: 15000 ns (5000 ns per operation)  
execute\_stream: 791 ns (263.667 ns per operation)  
CPP: 792 ns (264 ns per operation)  
\-------------------------  
Factor \[execute\_r\_nodes/CPP\]: 41.2449 (execute\_r\_nodes is 4024.494949% slower than CPP)  
Factor \[execute\_stream/CPP\]: 0.998737 (execute\_stream is around the same as CPP)  
\-------------------------  
\-------------------------  
       \==WARM==  
\-------------------------  
execute\_r\_nodes: 23963.2 ns (7987.75 ns per operation)  
stream\_r\_nodes: 54.448 ns (18.1493 ns per operation)  
execute\_stream: 446.248 ns (148.749 ns per operation)  
CPP: 269.086 ns (89.6953 ns per operation)  
\-------------------------  
Factor \[execute\_r\_nodes/CPP\]: 89.0542 (execute\_r\_nodes is 8805.422058% slower than CPP)  
Factor \[execute\_stream/CPP\]: 1.65838 (execute\_stream is 65.838431% slower than CPP)  
\-------------------------  
\==DONE==

As you can see, the streaming approach can reach consistently 50% the speed of C++ in this current implementation.  
Now since then, in modern GDSL versions, I’ve transitioned to a different module style which loses performance on streaming, but that’s because I didn’t want to deal with ASAN yelling at me while trying to program demo modules (which is what this is benchmarking, not the compiler speed, I imagine that’s slow and can’t be bothered to profile it because I’m not concerned about it).

## **Explanation**

type sub\_num {  
    int sub\_value;  
}  
type num {  
    int a;   
    int b;  
    int c;  
    layout();  
}  
print("==START==");  
int a;  
int b;  
int c;  
double d;  
string name;  
layout();  
num one;  
num two;  
num three;  
one.a;  
This is a test script using the demo modules GDSL meant to showcase what could be made in it.  
Here's what the memory layout looks like when you instantiate three num types, properties of different instances are contiguous in memory, not properties within one instance  
Here is the debug output:  
\==START==  
COL: 0:50155-56-784 1:50155-56-808 2:50155-56-832   
4     \-------------  \-------------  \-------------  
   0:  50155-26-720   50155-06-656   50154-76-768   
COL: 0:50155-57-312   
8     \-------------  
   0:  50156-08-704   
COL: 0:50155-60-928   
24    \-------------  
   0:  50156-09-584   
COL: 0:50155-62-912 1:50155-62-936 2:50155-62-960   
\[Break here for clarity, it's this layout:   
string name;  
\- \> layout();\]  
4     \-------------  \-------------  \-------------  
   0:  50156-10-672   50156-09-808   50154-90-320   
COL: 0:50155-62-912 1:50155-62-936 2:50155-62-960   
\[Break here for clarity, it's this layout in the Num constructer:   
    int c;  
   \-\> layout();  
}\]  
4     \-------------  \-------------  \-------------  
   0:  50156-10-672   50156-09-808   50154-90-320   
   1:  50156-10-676   50156-09-812   50154-90-324   
COL: 0:50155-62-912 1:50155-62-936 2:50155-62-960   
\[Break here for clarity, it's this layout in the Num constructer:   
    int c;  
   \-\> layout();  
}\]  
4     \-------------  \-------------  \-------------  
   0:  50156-10-672   50156-09-808   50154-90-320   
   1:  50156-10-676   50156-09-812   50154-90-324   
   2:  50156-10-680   50156-09-816   50154-90-328   
\[Break here for clarity, it's this layout in the Num constructer:   
    int c;  
   \-\> layout();  
}\]  
\==DONE==

Each Layout() call is telling you the positions in memory of the variables. And the size.  
You can see how the num "objects" are being created. Nm one int a is 4 bytes from num two int b. This is so the values are all cache aligned across objects, so we get an ECS-like (or database) memory layout but with OOP syntax.

And here's the underlying module code for the assignment operator, which is particularly gnarly:

 size\_t assignment\_id \= reg::new\_type("ASSIGNMENT");  
    state\_is\_opp.put(assignment\_id,true);  
    token\_to\_opp.put(equals\_id,assignment\_id);  
    type\_precdence.put(assignment\_id,1);   
    size\_t t\_assignment\_id \= reg::new\_type("T\_ASSIGN");   
    t\_opp\_conversion.put(assignment\_id, t\_assignment\_id);  
    size\_t r\_assignment\_id \= reg::new\_type("R\_ASSIGNMENT");  
    r\_handlers.put(t\_assignment\_id, binary\_op\_handler(r\_assignment\_id));  
    exec\_handlers.put(r\_assignment\_id, \[\](exec\_context& ctx) \-\> g\_ptr\<r\_node\> {  
        execute\_r\_node(ctx.node-\>right, ctx.frame, ctx.index,ctx.sub\_index);  
        if(ctx.node-\>left-\>value.sub\_size\!=-1) { //Indexing write  
             if(ctx.node-\>left-\>right) {  
                execute\_r\_node(ctx.node-\>left-\>right,ctx.frame,ctx.index,ctx.sub\_index);  
                int index \= ctx.node-\>left-\>left-\>value.address;  
                int sub\_index \= ctx.node-\>left-\>right-\>value.get\<int\>();  
                int total\_size \= ctx.node-\>left-\>left-\>value.sub\_size;  
                int size \= ctx.node-\>left-\>left-\>value.size;  
                void\* array\_start \= ctx.node-\>left-\>left-\>in\_scope-\>get(index, ctx.index, total\_size);  
                void\* target\_element \= (char\*)array\_start \+ (sub\_index \* size);  
                memcpy(target\_element, ctx.node-\>right-\>value.data, size);  
            }  
            return ctx.node;  
        }  
        execute\_r\_node(ctx.node-\>left, ctx.frame, ctx.index,ctx.sub\_index);  
        if(ctx.node-\>left-\>slot\!=-1) { //Is an object  
            size\_t target\_index \= ctx.node-\>left-\>index\!=-1?ctx.node-\>left-\>index:(ctx.sub\_index==-1?ctx.index:ctx.sub\_index);  
            if(ctx.node-\>left-\>value.type \== GET\_TYPE(OBJECT)) {  
                //Add opperator overloading for \= here  
                size\_t r\_target\_index \= ctx.node-\>right-\>index\!=-1?ctx.node-\>right-\>index:(ctx.sub\_index==-1?ctx.index:ctx.sub\_index);  
                //print("FROM R: ",ctx.node-\>right-\>in\_frame-\>context-\>type\_name,":",target\_index,"-",ctx.node-\>right-\>slot," TO L: ",ctx.node-\>left-\>in\_frame-\>context-\>type\_name,":",r\_target\_index,"-",ctx.node-\>left-\>slot);  
                ctx.node-\>left-\>in\_frame-\>slots\[target\_index\]\[ctx.node-\>left-\>slot\] \= ctx.node-\>right-\>in\_frame-\>slots\[r\_target\_index\]\[ctx.node-\>right-\>slot\];  
            } else { //It's a property access or other refrence  
                ctx.node-\>left-\>in\_scope-\>set(  
                    ctx.node-\>left-\>value.address,   
                    ctx.node-\>left-\>in\_frame-\>slots\[target\_index\]\[ctx.node-\>left-\>slot\],   
                    ctx.node-\>right-\>value.size,  
                    ctx.node-\>right-\>value.data);  
            }  
        }  
        else {  
            size\_t target\_index \= ctx.node-\>left-\>index==-1?ctx.index:ctx.node-\>left-\>index;  
            if(\!ctx.node-\>left-\>in\_scope) print("r\_assignment::859 no scope on left opperand for assignment");  
            // print("T: ",target\_index," S: ",ctx.frame-\>context-\>type\_name," LS: ",ctx.node-\>left-\>in\_scope-\>type\_name);  
            // print("AD: ",ctx.node-\>left-\>value.address," ",ctx.node-\>right-\>value.size," ");  
            ctx.node-\>left-\>in\_scope-\>set(  
                ctx.node-\>left-\>value.address,   
                target\_index,   
                ctx.node-\>right-\>value.size,   
                ctx.node-\>right-\>value.data);  
        }
        return ctx.node;
   });

Working with index-based memory layout like this is quite a pain, and that’s part of the reason I moved on before trying to implement all the array operators to do benchmarks of the actual performance with cache aligned data.   
The rest of the demo modules can be explored here:  
[https://github.com/FirTheMouse/Golden/blob/main/Projects/GDSL/include/modules/standard.hpp](https://github.com/FirTheMouse/Golden/blob/main/Projects/GDSL/include/modules/standard.hpp)


