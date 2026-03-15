3/14/26

By 2002 the weight of the software ecosystem had already reached a tipping point of sorts. Each of the big open source languages of the time, Python, Java, C, didn’t share code, they each rebuilt the platform layers from scratch in their own ways and suffered from it, or made one platform (JVM). This was the world Chirs Lattner wrote his thesis in, a thesis he would go on to prove to Apple in 2005 once he was given the chance, and the rest is history. 

Since then, LLVM has stood as the king of backends, but like any king it has grown old, and there are many hopefuls rising to vie for the throne, like MLIR. That core though, one backend to rule them all, catches my attention in a very particular way.

I have an affinity for general systems, it's the mode in which I build, and I identified with Lattner’s feelings when reading his story. Yet there’s a peculiar disconnect behind the idea and the product: Lattner sought to solve the fragmentation problem, but ended up setting the stage for an even worse fragmentation that each contender is now inheriting.
And I wanted to write about why I think that.

Before I ever started working on anything adjacent to compilers, I knew of LLVM as an indispensable tool: the backend which ran the code I cherished so much. LLVM was the arbiter of how many mice I could have on screen for my games, or what depth my chess engine could minimax too, so I didn't have much reason to think of it. To the contrary, I respected LLVM as a monolith of technology I would probably never touch. The wisdom I heard every time I dipped my toes into that deeper technical domain is 'LLVM already solved the hard problems' and 'it would take years to build anything like it'.
I agree with the second statement now, and that isn't to LLVMs credit.
The smell started rising the day I compiled a subset of C in 1,500 lines. Orders of magnitude less optimized than LLVM, yes, but deep enough that I could see a particular shape rising from the murky depths of its codebase. This was made clearest when I looked to the efforts of others in compiling C and found skyscrapers hundreds of thousands of lines tall. This isn't a piece about those though, that will come soon enough, no, this is a piece about the basement running even deeper: LLVM.

You would think with Lattner's original vision that LLVM would be a reduction of lower floors rather than an extension of the subbasement, yet that wasn't how it had turned out. That made me curious enough to descend, and start sniffing for the source of that smell.
The first thing I saw was the lobby, practical, widely used, bustling with activity. It's as good a lobby as any structure could have, and it was inviting enough that I considered writing up the handlers to have a GDSL module which emits to LLVM. Yet as mentioned, LLVM runs a much, much deeper basement underneath that lobby.
On the surface, the fragmentation problem was solved. We have a nice foundation, we have the standard… but can you call a lobby a foundation? What really lies beneath?

Fragmentation.

Because LLVM didn’t solve the problem, it just built a solution and moved it high enough that going deeper or ascending was a painful exercise. The codebase is massive enough that approaching it from either end you get bogged down fast. Plenty of people come at it from the  bottom, tunneling through the basement after they’ve built their own stack machines and assembly systems, they tend to take the elevator up to the lobby when  they decide to build a real language. Plenty come at it from the top, content in the shiny lobby and curious about what’s supporting it, though they see the pipes and wires and miles of concrete and conclude it's probably nothing they could understand, and besides: there’s skyscrapers to build. 

I never entered the basement, I rolled down its side. To my eyes, LLVM isn’t the basement of a skyscraper with a pretty lobby, it's a great hill of concrete that the skyscraper is built on. As I’ve tumbled down it I’ve had moments facing the sky and moments facing the ground, at once it's a marvel of engineering, and also a questionable thing to exist. That was until I hit the bottom about 5 days ago, when I could run C in 1,500 lines and looked at emitting to native, and realized that the hill is not the base. There is a network of bunkers that run much, much deeper beneath: the OS, layers you don’t explore or tumble down, you just have to go around. The structure is too dense to accommodate a person, and even if there were space to navigate it would be so immeasurably tangled that it would be a futile exercise. 

Though that could just be my perspective as a person who’s only just hit the top.
From here what I notice is that the hill was built to solve the problem of its own foundations. It saw the layer of bunkers and realized they required every structure above to build to their varying entry points or not get built at all, so it decided to construct itself over those entry points, build a nice lobby, and declare victory. Yet the bunkers still remain, the buildings have just gone further from them.

But why does it matter?

Because when you always build on concrete you always build with concrete.
You internalize the shape of things must be dense, that complexity is acceptable, that anything serious must require a team and a PhD. Nobody asks if it's possible to compile C in 1500 lines because that's akin to asking what if there were soil beneath our feet. And if you can't even imagine soil, how are you supposed to see trees?

And so, I search for holes. A place for me to get deeper if I can and do my best to make an answer to the question of fragmentation, because the answer won’t be found atop this field of bunkers.

