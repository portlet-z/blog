- Today we're gonna be talking all about structured bindings in C++ this is specific to C++ 17. structured bindings are a new feature that let us deal with multiple return values a little bit better
- now I did make a video about how to deal with multiple return values in C++ check out that video if you haven't already I'll have it linked up there
- and this is kind of extending upon that with kind of a new way of how we can deal with this specifically how we can deal with tuples and pairs and returning things like that because structured bindings just let us kind of simplify our code make it a lot cleaner that what it was in the past 
- in that video about how to deal with multiple values in C++ I did specifically mention that I like structs and I like to return basically instances of structs which contain the members of data that I actually want that's how I personally like dealing with multiple  returns 
- guys that could have potentially changed with this introduction of structured bindings and in fact it has changed because over the past like year or two really in my own code I've kind of noticed that I've been using tuples and like pairs and tires and I've been using that kind of stuff basically having multiple return values
-  so I'd like a tuple I've been using that a lot more often because structured bindings help me actually make my code still manageable because I used to absolutely hate what it was like before in fact let's take a look at what it was like before 
- so I'm gonna write a very simple example just running it from scratch here to make it really really obvious and what I'll do here is just write a function that creates a person now a person is a nice example because you might want to store more details about a person than just for example their name in this case we'll deal with their name and their age
- so we'll write a function called create person we'll need to set a return type which we will set to std::tuple and this is gonna contain the actual kind of multiple return values that we want to deal with in this case we'll deal with a string for the name of the person and then an integer for their age and we'll include string up here as well and of course we don't need to actually write age just the type 
- so we have a string and int over here we'll simply return in this case will be nice and simple we'll just simply return as the name and then an age watch for my age which is the 24 
- so now we have basically a mechanism to return two different types of data a string and an int without having to like create a struct or anything like that or just like pass parameters by reference or like as a pointer nice and simple and clean of course in this case you could just use a pair because there are two variables that this actual data structure is holding however with the tuple you can of course expand this to containers basically as many values as you want 
- so the in previous versions of C++ you had a few options for how you would deal with this well write some code to kind of show that 
- so I'll write std::tuple string and you can see already the type is a little bit messy but we'll have our person here then we'll set that equal to create person 
- now simplify this a little bit more you can just use order here in fact that's what I probably would do in this case but now we have the task of accessing the data and this is where it gets really annoying
- so basically the way that you can't just do person dot you know name as if it was struct you have to use std get and then as a template argument the index of the data you want to get 
- so zero for example would return the name the first variable which is a string and then if I used one here that would return the age 
- so basically to get this and you put the actual variable as a parameter here so to get the actual string which is the name I'd have to write code like this or you can use auto of course and then the age as well would look a little bit cryptic and it would basically be the same with std::get one and this is a little bit well it's just not nice 
- I probably would never use a triple for this case now there is something called std::tie which is a little bit nicer than this you still have to actually create true variables so name and age over here but what you can actually do is just pass them in here they get passed in by a reference name an age and then just set this equal to create person and this is a little bit nicer than all of this of course we don't have our actual person variable because we don't there is no person right it's not like it's a struct it's not like it's own type 
- it is just a container a tuple in this case that holds what we want which is a string and an int so this does look a bit nicer and I could probably be a little bit more inclined to do it this way however it still takes up like three lines of code it still does not look nice I still probably would use a struct and just basically create a person 
- so that then I could return that and then obviously just access it by person name and person till age as I showed in that C++ video about multiple return types
- now this is the part that you've been waiting for this is where C++ 17 brings us a new feature called structured bindings that solves all these problems and makes our code look really nice instead of doing all of this I will raise my person up here instead of doing all this we still keep this as a tuple we don't need std::tie or anything like that all we need to do we can get rid of these two lines of code all we need to do is use the word auto  followed by the names that we give to our variable so we can give these any names we want whatever so name and age would be good in this case and then we just set it equal to create person and that is it 
- this is a string  this is int it does everything for us perfectly and if we wanted to like we could print this we could do anything we want with these variables because now they're totally just in this scope and totally accessible to us
- now keep in mind this feature is only in C++ 17 and newer versions of C++ so if this does not compile for you make sure you're not compiling once a C++14 or 11 or anything like that make sure you switch your C++ version to C++17 specifically in visual studio we can go over here into properties and then go into C/C++ language and then make sure that our language standard is actually set to C++ 17 standard if it's set on default it may not work so for example if I do switch it to default and I try  and compile this code you see that it will not compile we will get an error saying that name is undeclared identifier and in fact we will get an error that tells us what we need to do it says the language features structured bindings requires compiler flag C++ 17  

```c++
std::tuple<std::string, int> createPerson() {
	return { "portlet", 31 };
}

int main() {
	auto [a, b] = createPerson();
	std::cout << a << std::endl;
	std::cout << b << std::endl;
	std::cin.get();
}
```

- so to switch it to that just make sure you go into your properties and set your sample stuff language standard to be C++ 17 now of course if you're using a different IDE or a different build system than visual studio like I am in this case then you'll have to do the equivalent for that but save us C++ 17 is what you need to have structured bindings to use structured bindings like this 
- I want to show you guys one more example so what I've got there is some code from the OpenGL series specifically the shader compilation code I did talk about the possibility of using tuples in that video as well which I might link in the top right corner but basically what we ended up doing was we need to pass a shader which we took in a file path floor and then we just read that file and the result of this function was splitting up that shader file into two different strings based on the shader  type and so we did to support that is we returned a struct as you can see if we look at what that struct is we have a vertex source and a fragment source 
- so what I'm going to do is change this to use structured bindings so that we don't need this struct at all so all delete the struct will come over here into our path shader we're not going to use a pair here we'll use a tuple the reason is that this theoretically could support more types than just fragment and vertex shaders which is what we've got there right now
- so what I'll do is just use the triple again you could use a pair then later change it into tuple it shouldn't break any code but just giving a reason behind why I'm using to build parent table in this case identical results 
- okay so we'll come over here and we'll also include that tuple and now I will hop over into my a C++ code I will change this to be the same thing the cool thing is that this return statement does not change at all because we still construct the tuple in exactly the same way but if we come over here into where we actually use this code which is up here instead of using this shader program source type and then doing source type vertex or source table fragments source instead of that we can now just use auto by vertex source and fragment source and then just set that equal to pass header and then into the create a function this also disappears and just simply becomes a vertex source and fragments source
- now in this case you can see that this does not compile because we need to switch this to be C++ 17 which currently is not so let's just do that here it is language standard and we'll set that to C++ 17 now you can see the error goes away and this is what we're left with so really simple really clean code if we go back to our header file you can see we got rid of that shader source struct altogether we know how one less type floating around which is a lot cleaner a lot easier because this is really the only case in our code where we actually want to use that shader saw struct this was used all around our code base you wouldn't want to wear this into a tuple and use structured bindings like this but in this case it just makes so much sense why have a type that you only use once that I don't really like the idea of that because it just clutters up your code base you have an extra type floating around not necessary 
- you can just use it structured bindings and tuples or pairs in this case 
