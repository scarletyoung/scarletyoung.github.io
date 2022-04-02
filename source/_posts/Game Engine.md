# Overview
* Entry Point: Entry point essentially when we launched our application or our game that's been made with this engine what happens. What controls where the main function is what controls the code it get executed with the main function. Is that something the client controls or the game controls or is it something that the engine actually controls and may be we kind of implement by either a macro or it gets imported somehow what happens with that. if we break this down to primitive building blocks when we launch our game or when we launch our engine what happens, what controls
* application layout: the application layout is just really a section of our code that deals with application kind of life cycle events and stuff like that. so for example what keeps the application opening and rendering frames. what keeps the time going forward. what execute all of the code that our game wants to execute. what about events. what about things like windows being resized or like closed or input event like mouse and keyboard. all of this kind of stuff is dealt with in the application layout. We need a wayt to basically run our game or run our engine as an actual application on whatever platform. that's what the application laryer kind of fits in
* Window layout: 
  * input and events: input kind of gets put into events and our event manager will handle inputs.
* renderer
* render api abstraction
* debugging support: when we construct any kind of application we need good ways to actually see what's going on and debugg it and solve our problems and build something, so it's something like a logging system. We needo to have it so that we can log any kind of data ypes that we have really easily and with a little bit to files and all that stuff. what about performance. In order to work that out we need some kind of profiling system. we want to be able to basically have our application potentially run in the special mode that is outside of like visual studio settings like we want our application to actually do itself so that we can run it on any platform and not have to worrying about using certain tools that are available on certain platforms.
* scripting languages
* memory systems
* entity component system(ECS)
* physics solution
* file io, virtual file system
* build system

# Event

# Precompiled header
What precompiled headers actually do is they give you an opportunity to grab a bunch of header files and convert them into essentially a kind of compiled format that the compiler can then use instead of having to read those header files over and over again.

Do not put frequently changing files into that precompiled header.

# Layer
Whenever you kind of want to draw something on the screen, the layers determine what the order things are drawn in. Layers aren't just applicable to graphics. Layers in game engine also are applicable to events and things like update logic