welcome back everybody um to the second lecture of 260 or 162 why i'm getting ahead of myself here um so what i would like to do today is 

dive right into the material and um let's try to keep comments during the actual lecture in the chat as actual questions and 

let's see what we can do so 

as you remember from last time 

we we're talking about what an operating system is and um you know i think i hope i kind of mentioned by the end of the lecture that it's really hard to say exactly what it is because not everybody agrees so we could kind of ask what it does and that's what we did here we talked about how an operating system acts as a referee illusionist and glue where the referee is managing protections on resources we'll talk a lot about that as term goes on illusionist is the 

the notion that we're gonna somehow make it look like we have a really clean set of resources that are much better than the axle ones are and 

virtual machine technologies help in this in some sense and then the glue is kind of a set of common services that basically make writing programs on top of an operating system much easier and things that you're very familiar with are things like windowing systems and the file system so if you also 

recall we started talking about for instance what the os might do for protection and what you see here is an example of two processes we're going to say more about them as we go on today 

that each think they have a machine that has all of the system resources to themselves and has a set of virtual resources like threads address spaces files and sockets that they can utilize any way they want and we can have more than one of these running at the same time on top of the same physical hardware and that's the job of the operating system and one of the things that we did talk about 

which we're going to bring up again today is this notion that program 2 which is in green here is not allowed to talk or otherwise observe modify process one's state not allowed to modify the operating system in ways that aren't allowed not allowed to modify storage unless they're allowed so this is part of the 

referee aspect of operating systems and we kind of said if process two tries to do any of these things then typically 

the operating system will object and you might get a segmentation fault 

with core dump or something like that where the process is essentially killed off so um the other thing that we started to talk about is that the the world of hardware is very complex and it's for very many good reasons and that world of complexity needs to be managed in a way so that we can still write programs that function properly okay and the um a good question that's in the chat right now was back on this previous slide of what a second instance of the same program get its 

get a separate process and the answer is yes so if you recall a process was an instantiation of a program and you can have multiple instantiations of the same program so um among the examples of complexity i just wanted to show you here we kind of talked about the sky lake 

processor series 

last time briefly and you can see that there's um the core 

chip itself is um directly connected to really high bandwidth dram channels and 

high speed graphics and so on and then there is this 

interface the direct media interface to what's often um used to be called the south bridge but it's basically the the 

chip that connects to all the rest of the i o and off of that we can have things like high-speed io devices for pci express we can have disks we can have slow i o through usb we can have ethernet hd audio pcie drives raid smart connect whatever and then there's an even slow interface off of that that gives us bios and all sorts of interesting things so really the reason that operating systems are so crucial is they provide a way to take this complexity and manage it okay now a slide i didn't really get a lot of time to talk about last time was this one which you can go to 

this link that i've got down the bottom here information is beautiful.net slash visualizations million lines of code and it kind of talks about millions of lines of code for different things that you're familiar with and if you look for instance one of the things that's a constant of the universe is for instance that going to a later version of something typically increases the size of that thing and the complexity of that thing tremendously so for instance from linux 2.2 to linux 3.1 if you notice that's a that's an increase of maybe a factor of six or more and the other thing to look at is things like cars which we take for granted are getting very complicated in terms of millions of lines of code so this is almost 100 million lines of code in a modern car that 

boy when you're traveling at highway speeds you want to make sure there aren't any bugs in that okay so we have a lot of complexity to 

deal with and of course 

yeah i have no idea how many lines of code in the 747 that's a really good question i would say a lot especially especially the modern systems which are essentially flown by computer pretty much if you think about ways in which the complexity leaks into the operating system 

what you see for instance is that third-party device drivers which are 

written by companies other than the os provider are often places where bugs happen and are the reason for a high fraction of crashes okay and um you know you buy a device it gives you a third-party device driver which you then install that device driver wasn't necessarily 

written all that carefully although microsoft over the years has come up with vetting processes to try to make things better so as apple so as the linux folks but you could think of a device driver as a reason to provide a nice clean interface and ironically that device driver interface is one of the things that causes things to crash holes in security models we'll talk a little bit later in the term about for instance spectre and meltdown these are the two symbols for it but 

all of a sudden in late 2017 early 2018 everything people thought they knew about securing data in a kernel turned out to be wrong because of the way that people were designing processors to do speculative execution and basically you could extract data directly from kernel space in many instances which is an issue the version skew on libraries can cause problems and that's one of the reasons that docker is so helpful and we'll talk more about that as we go on and then of course there's the invariable data breaches attacks timing channels 

etc... okay and you know the question in the 

one of the questions in the chat here is sort of why are device drive drivers particularly vulnerable and it's really that 

they are the the part of the system that touches the 

most complicated hardware and 

basically they're trying to put a clean interface up to the software but what's inside of them is potentially very complicated okay and there's a lot of interesting things that you can get just by googling for instance and i encourage you all to do that we'll talk more about device drivers when we get later in the term so the operating system is really trying to 

help abstract the underlying hardware and tame complexity all right and so you could think of there's hardware 

underneath the operating system is in between 

to provide a clean abstraction which we'll even call a virtual machine abstraction to the operating system okay now the question about how do we quantify 

reliability and so on there have been a number of attempts at that it's been hard but if you actually look at people that have measured the root causes of a lot of crashes um something upwards of 50 or 60 percent of them at one point in time were actually attributable to bugs in opera in device drivers which is 

pretty spectacular so the way we 

deal with this abstraction 

mechanism here or this virtualization mechanism is again we're providing various resources that are better than the hardware versions so instead of processors we're going to provide threads talk about that today for the first time instead of memory which is a bunch of dram pieces we're going to provide address spaces instead of disks or ssds which have blocks we're going to produce a file system instead of networks we're going to have a nice clean socket interface instead of machines 

we're going to have processes all right and this is going to be a an ongoing discussion throughout the next several weeks where we talk about how the operating system virtualizes the hardware pieces to give you a much cleaner environment the bios was asked about in the chat is typically a way of providing a set of standardized services on top of hardware and part of the bios is a legacy to old days in ibm pcs but some of it also provides firmware that can get updated and help the hardware be a little bit more reliable thereby making the operating system's job better okay and yes device drivers run in supervisor mode which is one of the reasons except for micro kernels which we'll talk about later in the term as well so the operating system as an illusionist is really part of our topic today we're going to talk about the four 

interesting 

functions of the operating system that really are leading to this illusionist idea and we're mostly going to work on the thread and process 

concepts today okay and um basically as an illusionist the os is going to try to remove hardware software quirks as a way of fighting complexity and optimize for convenience utilization and reliability to help the programmer all right and for any os area you pick it file systems virtual memory networking scheduling etc you can ask the questions of what hardware interface do you have to handle and what software interface are you going to provide and oftentimes the hardware interface talks about a set of mechanisms that the operating system exploits to provide a set of 

clean mechanisms and policies up to software and we'll use that terminology as we go throughout the term okay and um yes it is true that complexity is a very hard thing to measure and to 

talk about in a quantitative sense but certainly you know what it means in a qualitative sense and it's that qualitative sense that tends to get in the way of people knowing that their systems are going to function properly now so today we're going to basically talk about four fundamental os concepts we're going to start with talking about what a thread is and a thread is going to fully describe a program state or a linear execution context it's going to have a program counter registers execution flags a stack etc this is going to be very familiar hopefully to all of you 

based on 61c and then we're going to now then move off into address spaces either with or without translation which are a set of memory addresses accessible to the program and we're going to talk about how with the right mechanism we can provide a much cleaner behavior than the underlying hardware and then we're going to introduce what processes are and finally we're going to talk about a particularly important hardware mechanism for the early parts of this class which is dual mode operation which is the fact that a typical processor has at least two different modes which we may loosely call kernel mode and user mode and we exploit that to give us our better virtual machine behavior okay yes so um moving forward now so what's the bottom line we're going to run programs and that 

we're going to learn how to write them and compile them so you guys get to do that 

right away with 

with 

homework zero and um project zero starting tomorrow and then once they've been 

written then we're going to talk about how things get loaded into memory okay after their executables pulled off the file system it's loaded into memory we're going to talk about the stack and the heap getting put together for that particular process and then we're going to transfer control which is really means the program counter of the processor is going to be pointing at instructions in the user code of that process and then the execution will start okay and 

and then of course the operating system is going to provide services like file system and so on to the program and 

and it needs to do all of these things while protecting the os and the process from other processes okay and other users all right so fred's 

can be thought of in the following way and 

we'll talk a little bit later about 

threads and their heaps but if you look um for instance here back in 61c you got to learn about processors and the processor was something that started out with having a program counter as you recall and a memory that it could read and in that memory was a set of instructions okay and so that program counter would point into the memory and allow the processor to fetch the next instruction if you all remember so we'd pull the instruction in from memory we would decode it and then we would feed it to the execution pipeline which 

the one that we often talk about 61c is the five execution pipeline for a risk style processor and after things were decoded they would feed a set of registers and enalu to do actual operations and 

execute as desired and at that point you'd go on to the next 

instruction and so on and increment the program counter okay and so um this is hopefully familiar to everyone from 61c and if it isn't i would suggest that you guys go back and review a bit but let's talk a little bit about our virtualized version of what we said there so our first os concept is going to be a thread of control and a thread is really a single unique execution context and it's got a program counter registers execution flag stack memory state and so now all of a sudden you're going to say well wait a minute is that just what you had on the previous slide okay and if you look 

yes what you learned about was a very simple 

fetch execute cycle in 61c once we get to 

something we want to provide to other people to users in particular we need to virtualize it and so the thread is going to be like a virtualized version of your 61c processor and a thread is executing on the processor or core so by the way i'm going to intertwine processor and core until we 

can make that a little bit more clear later but it's executing when it's resident okay and the processor registers so we may have many threads but on a given core only one of them is resident and has control of the program counter and registers at any given time okay so what does resident mean let's just be very clear so the registers have the the context of the thread or the root state includes a program counter currently executing instruction the program counter is pointing at the next instruction in memory and all the instructions are stored in memory the resident state includes intermediate values for ongoing computations so typically once you get into pipelining which you started to learn about 61c there's a lot of pipeline state involved in an ongoing execution if you're really interested in that i would highly suggest something like 151 or 152 where you learn a lot more about interesting pipelines and speculative execution we'll be talking a little bit about that throughout the term but that's more of a hardware architecture class but um resident also means that there's a stack pointer that has a pointer into memory which is the top of the stack and um pretty much the rest of the thread is in memory so there's some things in the registers that and the rest is in memory okay and you'll see how that looks in a second so a thread is suspended or no longer executing when it states not loaded in registers okay so it's kind of the opposite of resident and at that point the processor state is pointed at some other thread so the thread that's that's suspended is actually sitting in memory and 

not yet 

executing or not executing at all while something else is executing so 

program counter is not pointing at the next execution from this thread because it's pointing at the execution of the current thread okay now 

here is another view of what happens during execution this is another kind of 61c view if you look here here is 

the set of addresses which we're going to call an address space a little bit later in the arc 

in the lecture that um from 0 to 2 to the 32nd minus one and what it has in it is a set of instructions that are going to be executed and what you see in pink here is your processor okay and um let's hold off on questions about where the state's stored when the thread's not running the way a thread is different from a process if you can give me a few more slides we'll get to that as well but it's basically the process has a protection state associated with it so if you look at the set of registers and the pipeline that's the processor and this might be the currently running thread which means our execution sequence fetches an instruction at the program counter decodes it executes it writes it back to registers grabs the next instruction and repeats so this is a wash and repeat kind of scenario and so for instance the program counter might start at instruction zero and then goes to one and two three and four as we're going and this in essence is what it means to execute okay this is the the basic 

von neumann machine that we're all very familiar with 

that you learn about in 61c and that we're going to take for granted because we're going to put an operating system on top of it the one thing that's going to be a little different from 

what you're used to is rather than risk 5 which is kind of what they do in 61c we're going to use a more common processor called an x86 

which is the intel processor and probably all of you who have laptops these days all have x86s on them um i understand that 

apple is basically punting the x86 in some of the upcoming generations but we're probably all using the same processor and the set of registers are a little different from the risk 5 processors you're used to so there's smaller number of execution registers but there's also a lot of other things like segment registers and so on which we'll talk about over time but if you notice on the left here we have risk 5 which had say 32 registers associated with it on the right we have x86 which has a much smaller number of registers that you can actually execute on and then a bunch of other state now the question about what an execution flag is came up in the chat and a lot of different processors have the following 

associated with them if you subtract two 

registers to get a third not only does it give you the result of that subtraction but then a set of flags gets set like did when you subtracted those two registers was the result zero so that's like the zero flag or was it greater than or less than zero so there might be a greater than flag so those flags are then subsequently things that you can actually make branches on branch decisions on so you might branch of equal and the way that's going to work is with an execution flag so um take back take a look in some of the supporting things that we have for you on the resources page and i believe in a section maybe on friday they'll talk a little bit more about the x86 as well but 

you're going to get very familiar with x86 okay so the question is are execution flags like the control logic from 61c the answer is no so think of 

execution flags are like a bunch of little one-bit registers that hold some of the comparative results of what you just did okay so they're they're they're tiny result registers and you can save and restore them 

during a contact switch on certain processors and if you look here see the e flags so those for instance are some of the flag registers that represent the results of execution so how do we get the illusion of multiple processors we talked last time about you know doing a psa ux or some other 

task manager on your laptop and 

if you look you'll find that there are hundreds of processes that are just running 

mostly sleeping but they're all available on your current processor and so how does that work so for the next 

i will say couple of weeks let's mostly assume that a physical processor has only one core on it or one thread of execution in the hardware at any given time and we will we will um graduate to multi-core processors a little bit later but for now what we've got here is we want to have multiple cpus or the illusion of multiple cpus running at the same time so we can have multiple threads running at the same time we're going to have them all share the same memory so that the programmer's view is well i just have a bunch of things running and they all share memory okay um and the 

question is kind of how do we get the illusion here and this is not complicated it's kind of what you would think we're going to multiplex that hardware in time okay so threads are virtual cores and 

what i show you here is assuming again for a moment that we have only one processor or one core in the system then the way we get the illusion of magenta cyan yellow running at the same time is we just multiplex we run from cyan's thread for a little while and then from magenta's thread for a little while then cyans then yellow and so on and we repeat and so over time we get this multiplexing of the same physical hardware okay and um the contents of a virtual core or thread is what well clearly each one of these virtual cpus needs to have a program counter and a stack pointer and 

all of the register state that we're used to if we were running that thread on a single processor in 61c okay so there's registers you might ask where is it the thread itself well if it's currently executing so for instance if we're in a period of time where magenta is running then it's in the processor itself and when it's not running it's saved in memory that state is called 

thread control block or tcb okay um now let's continue on this illusion for a moment here so consider this multiplex view and time so at t1 

vcpu one is running a t2 

cpu 

the blue one is running so what happened between one and two anybody want to hazard a guess so good so what happens is a contact switch is the high level answer um the low level answer would be some event okay so the os got to run somehow between 

pink and blue all right and what happened during that contact switch is we saved all of magenta's 

pc stack pointers all the other registers in their thread control block in memory we loaded the pc stack pointer et cetera from vcpu to and we returned to run the the cyan one okay so um interesting questions here that are coming up so first of all one question was does each 

thread here get its own cash and the answer is no okay so typically in general it's no typically 

there's one cache per core and so 

they're all kind of sharing the same cache so as you could imagine if you switch too quickly then nobody gets advantage of the cache okay and um yes the cache or the tlb 

in a primitive processor has to be flushed when you switch more advanced ones it doesn't we'll talk more about that as it goes on 

the cache itself is typically in physical space and you're switching from one thread to another 

you just change page tables and so you don't actually have to flush the cache and we'll get we'll get into that you guys are way ahead of me on this um another question is how long does this take well this can take 

something of the order of a few microseconds and um you want to make sure that the time to switch isn't 

so long that you're spending most of your time switching okay that would be a thrashing scenario that could be pretty bad okay um so the other question which is great you guys are on top of things wouldn't it be better to say run the magenta one to completion and then pick up the blue one so that would be a yes on efficiency but not so great on responsiveness okay because the the poor task that's trying to run in that cyan or blue thread wouldn't get to run for a very long period of time potentially okay so we're going to talk about those issues when we get into scheduling okay so you can already see how you're all asking the right questions there's some very interesting ones here okay but let's move a little forward here um what triggered the switch well we've already said things like a timer went off or a voluntary yield we'll learn about that 

very soon where 

the magenta one maybe decided to ask the operating system to do some io at which point the os said oh okay let's 

let's schedule somebody else okay and 

the question about how many registers there are is going to depend vastly on which processor you've got so yes there are 32 integer registers on a risk five which you guys are used to and yes there are some floating point ones as well on an x86 there's a much smaller number of registers and so when you don't have registers in the processor you got to keep things in memory so you spend a lot of time going back and forth so good questions so um now that we've started talking about how to get the illusion of multiple things we can start looking at 

what this model gives us and the model gives us the following we may have a bunch of memory that's in blue here and we could think of each one of these virtual processes you know green yellow orange has their own stack and heap and data and code and they're all laid out in memory somehow and what we need to do is somehow keep track of where everything is okay so the thread control block is that where everything is so when we switch from green to yellow the first thing we're going to do is save out all of green's registers into its thread control block which is by the way in part of the kernel memory which i'm not showing here okay the question about in-flight instructions that's in the chat's an interesting one so typically what happens when you get an interrupt is you end up flushing the pipeline so mostly in-flight state is all squashed when you switch okay we'll talk more about that a little later too where are the tcb stored they're stored in memory for now we're going to say they're stored in the kernel i want to say for now because we're going to talk about user level threads and some pretty interesting things in a couple of weeks excuse me but for now let's assume they're in the kernel and if you're you know you're start working on pintos which by the way you are we're going to release project zero tomorrow 

you should take a look at thread.h and thread.c right away you'll start to see how it is that pintos which is the operating system we're using for the projects implements threads okay all right so let's talk about some administrivia you should be working on homework zero okay it's due thursday already okay 

and you know the um the reason for homework zero is really to make sure that you have 

experimented with everything and you're ready to go so you get to experiment with gdb you get to experiment with compiling you get to work on your tools okay you get to learn about git if you're not sure about it you get your virtual machines up and running okay and so um we're gonna have project zero up tomorrow i know originally it wasn't up until thursday but we're pushing that a little forward project zero is a chance for you to really get going on the pintos projects and it's intended to be done on your own so do not do this with potential partners do this on your own and it's really about everybody who's going to participate in a group learning some basic things about how to run the projects okay and so again i suggest you get moving on that right away as well i did want to say something about slip days on projects and homeworks this term because of the you know because of the virtual nature of this class and things are complicated and difficult to get moving we're 

we're upping the number of slip days to four for both homeworks and projects 

but when you run out of slip days and you don't get any 

credit for things that you're slipping on so i would suggest that you bank okay 

the bank your slip days don't use them up right away um because you may want them later in the term okay tomorrow is 

an optional review session for c and 

actually i think we're billing it as for a bit of c plus plus as well 

there's a zoom link that's going to be announced um it may be already up on piazza 

it will probably record it um but i would consider attending 

and in that we've got people are gonna go over some of the basic things about c that you're gonna wanna know okay 

c plus plus is not really required for this class 

in answer to the question that's in the in the chat there you're really going to use c okay but 

it doesn't hurt to look at what we've got for c plus plus it well as well not really required means that the the work you're doing in pentos is in c um and 

friday that is four days from now is drop date okay so it's an early drop date class and it's very hard to drop afterwards so if you're not interested in the class you should 

drop sooner rather than later so that people can be pulled off the wait list okay and the thing that you need to do is if you have friends who are either on the wait list or in class but they haven't been doing any work there are in danger of being put into the class without them knowing okay and you may you may think that that's ridiculous but it happens every term somebody wasn't paying attention and they end up three quarters of the way through the term and discover that they're in the class and they can't drop okay it's very hard to drop late 

without burning one of your you know your one and only late drop date so for late drop class so 

just try to make a decision on that okay any questions on that part okay it is true that berkeley does not 

have a mainstream c plus plus class 

in the computer science department there's lots of great languages out there 

and so 

somebody who knows c 

is at least i would say a third of the way to c plus plus 

but 

you may end up learning c plus on the fly for other classes like 184 for instance okay so just to remind you from last time you know this virtual class that we're in here is challenging and 

for everybody okay and things are um considerably different 

starting off remote not even starting off 

physically and so that means that we've got to figure out how to re-establish 

the people interactions and collaborations that we would have if we were in person okay um how do you recover collaboration without direct interaction that's going to be challenging and so i'm asking everybody here i'm putting out a plea to do your best to talk to people more than you would in a real term okay you gotta have more meetings 

drink coffee with your friends on zoom more often or with your 

groupmates okay this is important um and you gotta figure out how to bring people along virtually okay it's very easy in this world where um i heard umesh bazarani described what we're doing right now as we flatten the world graph okay so everybody's equidistant from everybody else and as a result nobody's close to anybody okay yes i've become a flat earther for 

with respect to this class cameras as i've mentioned before are essential components of this class so what we're trying to do is 

make sure that people maintain their interactions okay and we're gonna need it for exams and discussion sessions design reviews et cetera okay and um have a camera plan to turn it on and let's try to keep that human interaction going okay um the 

we need to bring back personal interaction um humans are not good at interacting via text only you can kind of see what happens with twitter in the public life is is really not a great thing and so let's do everything with 

in person as in person as we can get with a camera interaction okay and 

you're gonna have required attendance at the discussion sessions design reviews et cetera with the camera turned up okay now 

the other thing i wanted to remind you guys of is the collaboration policy all right you gotta um you know if you're explaining a concept to somebody that's okay if you're discussing algorithms or testing strategies with other groups that can be okay if you're discussing maybe debugging approaches with other groups but kind of at an abstract level that's okay if you're a large allowed to do searching for generic algorithms like hash tables okay these are all okay what you're not allowed to do is 

share code or test cases with other groups okay and we track that okay we have a mechanism to compare people's code with other people's code from earlier terms and in the class and so on just don't do it copying or reading another group's code or test cases just don't do it copying or reading online code or test cases from prior years or other members of your group just don't do that okay 

helping somebody to debug in detail in another group don't do that either okay because what happens is um if you know you're helping somebody debug you're now not only looking at their code but you're importing your code kind of conceptually into their code and we have had situations where debugging essentially caused the person that was being helped code to to 

match against the other code and both groups got in trouble so just just say no okay um we're 

compare all project submissions against prior year submissions and online solutions and so this is it you know just just don't do it and also don't put a friend in a bad position by asking for help that they shouldn't give you okay all right good now are there any other administrivia questions nope okay um now let me see there was a question oh there was a good question 

in the in the chat from before i started the break which was you know why not 

just kill off one stage of the pipeline at a time you know it turns out oops sorry that um it's very difficult to 

restart pipelines 

if you try to save some of the state and restart it that's called precise exceptions 

the question of precise exceptions if you can save part of the state and restore part of the state that's an imprecise exception turns out that gets complicated very rapidly it makes getting correct os code really hard so in general they don't do that okay and i will mention that a little bit later when we get into page fault hammer page fault as well okay now so 

if people could turn off their cameras during class that would be good please so 

the second os concept we're going to talk about today is address spaces okay and um the simple idea is that it's the set of accessible addresses and the state associated with them so if you think back to 61c you've got 

let's say a 32-bit address from zero up to fffffffffff and this is the view that a processor has of memory now that's not to say that there's 

dram in all of these spaces it's just that this is the processor's view of what addresses are available and so for a 32-bit processor by the way i'm gonna make you guys all know about powers of two so if you don't know them yet you should start learning them um but two to the 32nd for instance is four billion approximately okay 10 to the ninth to the 64 is 18 quadrillion so that's a lot of addresses okay but um if you think of the address space as all of the the potential places that the processors could go and then there are ones that actually are backed by dram then there's some state associated with them and the question might be well what can you do when you read or write to an address well perhaps it acts like regular memory or perhaps it ignores the write entirely or perhaps the system causes an i o operation to happen that's called memory mapped io or perhaps it causes an exception it's possible if you try to read or write somewhere in the middle between the stack and the heap that i'm showing you here and there's no memory assigned to that process you get a page fault okay or maybe the act of writing to memory communicates with another program okay so um there's a lot of 

a lot of possibilities here okay now 

so am i saying oh i meant quintillion there didn't i okay thanks for the catch 

so um in a picture i'll fix that slide by the way so in a picture the address space is kind of like this okay so here is your 61c processor registers okay the program counter points to some address and the stack pointer points to some address typically the bottom of the stack and um other registers might point to things in the heap or et cetera and the fact that the pc can point to an address and it can fetch from that address means that we can have a processor that actually executes an instruction at that address okay so whatever we come up with with our threading and protection model it's going to involve accessing the address space okay and so what's in the code segment um well it's in the code segment's code that's not too surprising what about the static data segment anybody have any idea what would be in the static data segment so many of you have started looking at gdb great static variables yup global variables et cetera things that are um explicitly declared rather than allocated with malik good yup string constants all of those things are typically in the static data and that's loaded at the point where the program's first loaded while the process is being created what's in the stack segment anybody remember what is on the stack yeah local variables okay we're going to go through this more but you should look back at 61c and recall what the stack's about right so the stack is when you do a recursive call to a function the variables that were of the previous function are pushed on the stack and then the stack pointer moves down and then when you return you 

pop them off the stack and stack pointer moves up so i also see locals that's correct so local variables the 

how do we allocate it well we'll talk more about that one of the things that's going to be a very cool thing the operating system can do once you've got virtual memory is you can start the stack off with just a couple of of pages and then as 

the stack tries to grow it's going to cause page faults and the operating system will then be able to add more physical memory to the stack okay and the same is true of the heap so the heap is when you allocate things with malik or so on or you do linked lists all of those things typically lay in the heap and the heap is als also starts out with less physical memory than maybe the program ultimately needs and as the program starts to grow you get page faults which will allocate things on the heap okay so you don't have to worry about having caught all that now but i'm just giving you some ideas okay what's in the heap segment well i already said that that's things that you have allocated with malik think of structures with pointers think of linked lists think of all sorts of things okay there's a rather amusing comment in the operating system that they are really convoluted magic um maybe on the other hand i'm hoping that by the end of the term you'll see that they're just very clean magic okay all right certainly validating parts of the operating system can be easier to validate than a compiler now so our previous discussion of threads is what i would call very simple multi-programming okay all of these vcpus share the same non-cpu resources the only thing we virtualized with our current threads are the registers the program counter the stack pointer nothing else so that means they all share all the rest of memory they all share i o devices they all share everything else okay and that could be an issue now the question that's on the the chat which i find interesting 

here right now is can they 

each assume they have infinite stack or heap 

that's a really tricky question that we'll have to 

defer a little bit more um for for a week or so but the short answer is that 

if they actually are threads and they're saying in the same address space then the threads can mess each other up by overwriting each other's stacks but that's actually 

sort of a design feature okay if on the other hand you don't want that to happen you put these in separate processes so hold on for the rest of the lecture on that part okay now the os's job is going to be making the virtualization be as true as possible given the resources and whether it's a process or not so um so if each thread can read or write every other thread's memory maybe their data maybe their security keys and so on could it overwrite the operating system well so far i haven't given you anything that would prevent you from overriding the operating system and 

back in the early days of personal computing okay we're talking about some of the original ibm pcs some of the original macintoshes which were these weird-looking square boxes 

the early days of windows definitely all had this problem okay that 

yes we provided the ability to have illusions of multiple threads at the same time but they were all in the same address space and they could overwrite each other okay so we want to do better okay so is this an unusable environment well depends on your definition of unusable it's certainly not very secure okay and it's not even very secure against your own bugs okay and we'll talk a lot about that but you'd like a system that 

when you put a buggy piece of software up and run it it doesn't crash everything else that would be kind of a minimum requirement i would say okay so this approach as i said was used the very early days of computing it's used on some embedded systems still macos you know windows 3-1 windows me a lot of those different ones basically had this view okay however it's risky you know one of my favorite things to do you'll find out my favorite number is pi um because why not i mean it's a great number 

but it goes on forever but what's interesting is you can imagine that the magenta one decides to compute the last digit of pi and it never gives up the cpu locks out the timer interrupts and now 

blue and yellow never run okay that's a system we do not want to have and that is a system we used to have okay i worked on windows 3 1 systems where you put the wrong application in there and all of a sudden you know everything locked up so that's rather undesirable so no protection so the operating system has to protect itself from user programs and there are lots of reasons for this right from a reliability standpoint 

compromising the operating system generally causes it to crash of course it does security you want to limit the scope of what malicious software can do privacy i want to limit each thread to the data it's supposed to access i don't want my cryptographic keys or my you know secrets to be leaked and also fairness i don't want a thread like that one that decided to compute the last digit of pi to be suddenly able to take all of the cpu at the expense of everybody else okay so there's lots of reasons for protection and the os must protect user programs from one another okay prevent threads owned by one user from impacting threads owned by another one um all right so let's see if we can do better okay so what can the hardware do to help the os protect itself from programs well here's a very simple idea in fact very simple that so simple that little tiny iot devices can do this with very few transistors and the idea is what i'm going to call basin bound and so what we're going to do is we're going to have two registers a base register and a bound register and what those two registers talk about is what part of memory is the yellow thread allowed to access okay what part of memory is the yellow thread allowed to access now that we're still going to call this by the way i've got this 

sorry zero at the top and ffff at the bottom i've swapped this for you guys but um the we are going to be able to put two addresses one in base and a length or an address inbound depending on how you do it and now we're gonna see whether we can limit yellow's span to just the 

those range of addresses okay we're still gonna say the address space is from zero to all f's it's just that a big chunk of that address space is not available to the yellow thing okay and so what happens here is a program address that fits somewhere in the the valid part of the program what really happens is the program has been relocated it's been loaded from disk and relocated to this portion of memory one zero zero and so now when the program starts executing it's working with program counters that are in the say one zero one zero range which is kind of right 

where the code is and hardware is going to do a quick comparison to say is this program counter greater than base or is it and is it less than bound okay and these are not physical 

excuse me these are still physical addresses these are not virtual addresses yet we'll get to that in a moment okay and this allocation size is 

challenging to change in this particular model okay because in order to get something bigger we might end up having to copy a lot of the yellow to some other part of memory that's bigger so you can see that this is just a very primitive and simple thing okay but what it does do is it gives us protection so the yellow code can run it could do all it wants inside the yellow part of the address space but it can't mess up the operating system or anybody other anybody else's code okay all right and 

whether base and bound are inclusive or not that's sort of a simple matter of whether you include equals or not so let's not worry about this obviously base is inclusive in the way i've shown it here um so the other thing is for every time we do a 

lookup we make sure that we're less than bound so it's not inclusive on the top in this particular figure and greater than or equal to base and if that's true we allow it to go through and if it's not true then we 

do something like 

kill the thread off or something okay now the address here has been translated if you look this is what it might look like on disk it's got you know it's code starts at address zero there's some static data after the code maybe there's a part of the heap or stack that's going to be in there once it's loaded but in some sense it looks like everything started zero and however when we load it into memory we relocate all the code so that it starts at address 1000 is runnable from that point and as a result things 

execute properly so this is a compiler based loader based relocation okay but it allows the os to protect and isolate okay it needs a relocating loader now this by the way was what a lot of early systems did is they did relocation okay and had some basin bound possibilities to work so for instance the early some of the early machines by cray had this this behavior okay notice also that we're using the program counter directly out of the the processor without train changing in any so we're not changing any of the the latency through transistors because we're not adding any 

extra translation overhead as well okay and the gray part up here might be the os yes now if you remember in 61 c we talked about relocation um so for instance if you do a jump and link to the printf routine um what that translates into is a relocatable code where maybe the gel op code is 

hard-coded but the printf address is not until things get actually loaded and then this gets filled out so this might be a relative address until the linker and loader pulls it into memory okay all right so we can do this with the loader okay but a number of you have started to ask more and more about virtual memory well here's another version of virtual memory that is actually well it's 

the previous one was a hardware feature because in hardware we're preventing 

the program when it's running in user mode or as a user from accessing the os so that's a hardware based check okay it's not software all right now the 

a slight variation on the basin bound is this one where we actually 

put a hardware adder in here okay and this hardware adder one way to think of this is that addresses are actually translated on the fly so now we take our yellow thing off disk and we load it into memory and it might still be at address 1000 but the difference is that the program is now the program counter is now 

executing as if it were operating in this 

code that starts from zero but in fact what happens is by adding the base address to the program counter we get a translated address that's now up in the space where yellow actually is okay all right so this particular 

version of this is 

it's very simple and it doesn't require page tables or complicated translation okay so this is a hardware relocation so on the fly the program counter which is operating as if we're in the yellow region we add a base to it and the thing we actually use to look up in dram is the 

new address the physical address that we get from this virtual address added to the base pointer okay and can the program touch the os once again no because if the program address goes negative we can catch that 

and so that would be below the base address and if it goes too too large above the bound then we would also be outside of yellow and so we basically protect 

the system against the yellow okay and so um once again we're still doing checks here now can i touch onto the programs no because the bound catches it so one way to get at this is also with segments okay so in the x86 code or x86 hardware we have segments like the code segment the stack segment etc which are hardware registers that have the basin bound coded in that segment so a code segment is something which has a physical starting point and a length and then the actual instruction pointer that's running is an offset inside of that segment so the the code segment is very much like this base inbound because we do this addition on the fly and checking for the 

the bound okay and the question about where does the base address how do we decide what the base address is how do we decide what the bound is well that's the os is basically doing a best fit of the current things it's trying to run into the existing memory now a different idea which they did bring up in 61c which everybody's clearly familiar with is this idea of address-based translation so notice that what we just did was a very primitive version of translation where we took every address coming out of the processor and we added to it a base and we checked it against the bound and that translation now is just add a base and check about but um the thing we could do that's even more sophisticated is we could take every address that comes out of the processor and go through some arbitrarily complicated translator and have it look up things in memory so 

if you look at 

how that might be so let's think for a moment what was the biggest issue with this so there's several issues not the least of which to grow the space for the yellow process or thread i haven't told you how to distinguish those yet we would have to um copy the yellow thing somewhere else okay and and what we're going to do is then when yellow finishes and goes away we've got a hole we've got a fill and so there's a serious fragmentation problem so we'll talk more about that in an upcoming lecture but what we're going to do is we're going to break the address space which is all of the dram into a bunch of equal sized chunks all pages are the same size so it's really easy to place each page in memory and the hardware is going to translate using a page table okay this is 61c okay special hardware registers are going to store a pointer to the page table and we're going to treat memory as a bunch of page size frames and we can put any page into any frame etc we're going to talk a lot more about this don't worry 

in upcoming lectures and this is another 61c idea okay but just roughly speaking so just from a high level don't worry about the details yet but if we take something like a program counter or registers that are pointing at memory we go through a page table to translate them that's going to give us a part of memory and now we can do interesting memory management okay now by the way i am the reason i'm covering all of these ideas is i'm giving you something to think about okay and we're gonna in in the upcoming lectures we are going to fill in details on this okay but this is some part of the story okay so instructions operate on virtual addresses um and these are you know instruction addresses load store addresses etc they get translated to a physical address and this is the dram so the processor is looking in one address space a dram and another okay and any page of the address space can be any page size frame and memory etc this is going to be great once you get a better handle on this because it's going to not have the same fragmentation problem that the base inbound did that we talked about earlier okay now the third os concept that we want to talk about today is a process and a process is really an execution environment with restricted rights so if you remember we talked about our simple virtual threads having this problem that everybody had access to everybody's memory well we started to note how mechanisms for translation might protect us okay and so the idea of a protected chunk of memory that's owned exclusively by an entity in an os that's called a process okay so the process has an execution environment with restricted rights and one or more threads okay so it's a restricted address space and one or more threads it owns some file descriptors and file system context we'll talk more about that as we go on and it's going to encapsulate one more or more threads for sharing in a unique environment okay and the good question 

is how do we how do we protect this there's a question on the chat which partially addresses this which is of course if you have two processes and their translations point to different physical memory then kind of by design they can't 

get at each other's data because there's no way for a processor running process a to even address the data in process b okay and we're gonna that's the advantage of translation we'll talk a little bit more about that a lot more about that in upcoming lectures so a process is an address space with one or more threads okay and the application program when you start it up typically executes as a process so we'll talk about fork and exec and how do we create processes but the upshot is we create a restricted address space environment and then we can run one or more threads in it and now all of a sudden we've got a process and that becomes our unit of protection 

for the first couple of weeks of the class okay and the page table is really going to translate between virtual addresses and physical ones and it can do both in a forward and a reverse fashion so we're going to talk about page tables in quite a bit of detail so don't worry about the details yet they're they're going to be coming just think of the high level idea of translating for now so why processes well because we're protected from each other and the os is protected from them and processes provide that memory protection abstraction okay and there's this fundamental trade-off between protection and efficiency so if you have a bunch of threads all in the same process then yes they can communicate really easily because they share the same memory so they can communicate by one of them writes in memory the other one reads from it but they can overwrite each other okay so there are times when you want to have high performance parallelism where you want a bunch of threads in a process but then when you want protection you want to limit the communication between processes so communication is intentionally harder between processes and that's where we get our protection from so here's a view of two different types of processes here's a single threaded one and a multi-threaded one so 

the only difference between these two is that the multi-threaded one has more than one thread running if you notice this box that i show here for the single threaded process for instance is the protected address space so everything that's going on inside here cannot be disturbed by something going inside in a different process and in this example since there's only one thread we only have sort of one stack and one heap and the code and data kind of live in there and this protected environment is great for the thread nobody can disturb it but if it needs to communicate it needs to figure out how to communicate outside of its process a multi-threaded process actually has a different stack for each thread because you need a stack to have a unique thread of execution it has a separate set of registers for the thread control block so that when we switch from thread to thread to thread to give that illusion of multi-processing we need to switch out the registers from the first thread so that we can load them back from the second thread okay so threads encapsulate concurrency the address space is the protection environment you could kind of think of threads as the active part in the address space is the protected part that may or may not help you but the address space which the protected address space is really going to keep buggy programs or malicious ones from impacting each other okay and why do this multi-threads per process well there are many reasons you might do this one is parallelism so if you actually have multiple cores it's possible that by having many threads in the same process you can have many things working on the same task at once you learned about parallelism in in some of your early classes like 61a the other reason might be concurrency so a good reason to have many threads in a process could be well most of them are sleeping but thread a deals with mouse input thread b deals with window movement threat c deals with io to disk or network or whatever and so that concurrency is a situation where most of the threads are sleeping most of the time but it's a much easier programming model to think of things as a thread that runs for a while does some i o going to sleep and then wakes up when the i o is done and as you get more familiar with threads and how they work you'll be able to figure out kind of you'll have a better idea why that's helpful um so the question of is there a parallelism efficiency advantage you'd get from a process that you couldn't get from a thread so keep in mind that a process has threads in it so think of the process is the container and the threads are the execution element okay and yes these are a little bit like fibers but a little more heavy weight okay so why do we need processes for reliability security privacy okay bugs can over only overwrite memory of a process that they're in malicious or compromised processes can't mess with other processes now of course if the operating system is compromised every all bets are off but we'll even talk about later in the term we'll talk about how to 

set up situations where even if the operating system is a bit compromised the 

the things that are running in it might still be secure okay mechanisms to give us protection and isolation well we already talked about the fact that we need some hardware mechanisms for address translation we showed you the very simplest which was an adder in the hardware we we hinted at something much more complicated like page tables or whatever but also we have to worry about well if we have page tables why can't process a change its own page tables to point at process b because that would destroy all of the protection okay and so that leads us to our fourth mechanism which is we need the hardware to support some privilege levels of some sort and so that's the idea of dual mode okay so hardware provides at least two modes okay and um the two modes are kernel mode and user mode okay or supervisor mode and 

certain operations end up being prohibited when you're running in user mode so when you're in user mode you can't for instance change which page table you're using that's something only the operating system in kernel mode can do you can't disable interrupts okay so that way a process that's decided it wants to compute the last digit of pi can't prevent other ones other processes from getting cpu time when the timer goes off okay you're also prevented from interacting directly with hardware et cetera thereby not being able to breach files on disk and now the question is what's our carefully controlled transitions between user mode and curl mode things like system calls interrupts exceptions okay and so you could think roughly speaking that we have user processes they make a system call into the kernel that's a transition from user mode to kernel mode that's very well controlled we'll talk more about that and then the kernel does a return back to user mode for the user to run okay and so this is a typical unix system structure monolithic unix system structure where 

kernel mode represents code that has secure access to all sorts of resources the it controls the hardware directly and then of course user mode is something that 

it's all of your programs and your libraries and so on and so user mode is your application but then it uses services from kernel mode and that's the operating system kernel okay so for instance here's an example where we've got hardware got kernel mode and user mode the hardware might execute or exec a new process okay and then later we'll exit and return to kernel mode okay a system call from user mode would go into kernel mode and then it might return later or an interrupt might cause user mode to go into the kernel which then might check out the hardware somehow and then eventually do a return from interrupt an exception which is something like you divided by zero or a page faults an exception might go into the kernel and then eventually return okay now there's additional layers of protection than just the two i talked about here and we'll when we get into talking about virtual machines and actually containers like docker and so on we'll talk about how to put more layers than just the two but for now we're dealing with dual mode okay now tying it all together 

we can tie it all together very simply okay um and give me 

so tying this all together 

is the following so if you notice here i have two processes a green and a yellow one okay and 

yeah that was like a page full and 

if you look at 

the os here is the gray code and if you notice our system mode right now is kernel mode so it's red and 

it's on and when we're in kernel mode with simple branch and bound what you see here is that there may be base and bound registers but they're being ignored because in system mode we have access to the full address space okay and let's take a look so the os is going to load a process okay what that really means is it's going to take this 

a register which we're going to call the user pc load it with a pointer to the code to the starting part of the yellow code and if you notice 

there's going to be a bound or an 

potential 

top of of that 

area and what we're gonna do is we load the the yellow off of disk we set up these registers okay but notice by the way since the os is running 

the pc is still pointing to gray not to yellow okay but what we're going to do is then we're going to execute a return from interrupt or return to user and that's going to start us running the yellow code now the question is why does stack grow up in these diagrams that's because i've got year 0 and fff reversed so the lower part of the date of the address is up top here in the higher part is 

on the bottom sorry about that um but notice right now the the kernel has full access to everything if we um now do a return to user what's going to happen is um that we're going to activate this yellow one okay so the privileged 

instruction is to set up these special registers like the basin bound registers are going to get set up and so on so notice we've set base to the beginning we've set bound to the end we've set up some special registers we've set up the user pc and we're going to do the return to user mode and that's going to basically do two things one it's going to take us out of system mode which is going to activate these base and bounds and it's going to cause the user's pc to be swapped in for the existing kernel pc and now all of a sudden after i do that voila we're running in user mode okay why do i say we're running in user mode the answer is that um right now because we're in system mode not we're not in system mode we're in user mode the base and bound are active and so the code that's running can't get out of this little container okay all right and um coming back so how does the kernel now switch so now we've got this guy running what do we do well we're going to have to take an interrupt of some sort and say switch to a different process okay so um the first question we have to ask before we 

figure out what the switching is involved is how do we return to the system all right and i showed you some opportunities there a little 

a moment ago but we have three so for instance system call 

is one where the process recov requests a system service that actually takes it into the kernel okay another is an interrupt okay this is the case where an asynchronous event like a timer goes off and takes us into the kernel and a third one is like a trap or an exception um it turns out that these could be examples where 

we get a page fault or where we divide by zero okay now the interesting question that's on 

an interesting question in chat which i don't have a lot of time to answer right now is 

was what if a program can needs to do something that can only be done in kernel mode the answer is you got to be really careful so one answer would be you can't you got to do only the things that are provided as apis from the kernel that's why the set of system calls is so important to make sure it's general enough for what you want to do the second answer gets much more interesting which is typically not something we talk about at this term in 162 but we could maybe and that's where we have a an interface for downloading specially checked code into the kernel to run in kernel mode 

in a way that it doesn't 

compromise the security but 

that's a pretty interesting topic for a different lecture so we could we could 

invoke any of these things system call interrupt or trap exception to get us into the kernel so that we can do a switch so let's assume we do an interrupt okay and these are all unprogrammed control transfers so how does this work we're going to talk a lot more about this in the next in another lecture but the way this works to have the interrupt interrupt into a well-defined part of the kernel is we're going to actually have that interrupt the timer interrupt look up in a table that we have put in there when the the os boots and pick the interrupt handler and that interrupt handler is now going to run and make a decision about whether it's time to switch from process a to process b and this is by the way the topic of lecture in a week or so okay so we're getting close to that those topics so um so let's continue here's our example the yellow code's running and if you notice the program counter again is in the yellow code and so on and how do we return to the system maybe an interrupt or io or other things we'll say an interrupt for this and what happens at that point is we now back in the kernel so notice that we're at system mode we're running the pc is the interrupt vector of the timer and we've got these registers from the yellow which 

have been saved as a result of going into the interrupt and so what we're going to do is we're going to save them off into the thread control block we're going to load from the thread control block for green okay and then here's by the way somewhere in the kernel is the yellow thread control block and then voila we return to user and now the green one's running so really this idea of swapping between processes and using dual mode execution is pretty much shown by this example i just showed you there which is you run for a while at user mode the timer goes off you save out the registers you load in other ones you return to user again and you just keep doing that back and forth all right and the question about how the current process knows if there's an interrupt the answer is it doesn't what happens is the interrupt goes into the kernel and saves all of the state and the yellow code in a way that when it's restored and starts executing again it doesn't know okay all right in a typical time period for how frequently the timer goes off 

it's typically like 10 or 100 microseconds between timer ticks okay now we're 

we're now officially out of time but i want to leave you with one more concept what if we want to run many programs so now we have this basic mechanism to switch between user processes in the kernel kernel can switch among the processes we can protect them but these are all kind of mechanisms without sort of policy right so what are some questions like how do we decide which one to run how do we represent user processes in the operating system how do we pack up the process and set it aside how do we get a stack and heap et cetera et cetera all of these are interesting things that we're going to cover um and you know aren't we wasting a lot of memory all of these things okay and um so there is a process control block just like the thread control block don't worry 

we'll get to that but that's where we saved the process state and inside of that will be the thread control blocks for all the threads that are there and then the scheduler is this interesting thing which some might argue this is the operating system which is every time or tick it says it looks at all the ready processes picks one runs it and then 

the next time or tick it runs the next one and so on and part of that process is unload and reload unload and reload with the some task called the scheduler selecting which is the right one based on some policies all right so we are done for today so in conclusion there are 

four fundamental os concepts we talked about today the execution context which is a thread okay this is what you learned about in 61c didn't call it a thread because it wasn't properly virtualized yet but it's basically something with program counter registers execution flags stack we talked about the address space is the visible part of the 

to a processor it's the visible part of the addresses and once we start adding translation in now we can make protected address spaces which are protected against other 

processes we talked about a process being a protected address address space with one or more threads and we talked about how the dual mode 

operation of the processor hardware is what allows us to multiplex processes together and give us a nice secure model all right so there we go that is 

in a nutshell a modern os so um that'll be the end of this class there'll be a final 

in several months and oh wait i'm just kidding i hope you guys have a great night and we will see you on wednesday ciao thank you
