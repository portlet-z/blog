- today we're going to be talking about what happens when we have data that may or may not be there a lot of the times we have like a function that for example returns data let's just say we're reading a file but what happens if the file could not be read what happens if we couldn't load the file if it's not present
- if the data just isn't in the format that we expect we still need to return something from the function now a lot of time the time in that particular scenario you might just return an empty string but that's not really that great because it doesn't make much sense I mean what if the file was just empty there should be a way for us to actually see if the data was present or if the  data wasn't present and that's where std::optional comes in
- so as to the optional is new in C++ 17 and it's gonna be like a part of like a three part miniseries in which we cover three different classes three different new types that C++ gives us to deal with data that either may or may not be there of is of a type that we're not sure about
- so let's jump in and take a little bit of a look at some code that could be written one way but now with std::optional we have a better way of dealing with data that may or may not be there 
- so this is going to be a super simple example I'm starting with absolutely no code at all and what I'm gonna do is write a little function which reads a file so this will basically be like read string from file  right or maybe we'll rename it to read file as string so this should return the entire file for example as a string so we'll take in a const std::string file path and then over here we'll just have an input file stream we'll take in the file path of what we actually want to read 
- I'll just quickly include <fstream> and yes I will make a video soon about how to read files and see if I suppose I know I haven't done that yet and now that we have our input file stream what we want to do is read it if it's valid if the file was open successfully or if it wasn't we have to handle that as well so very simply we can just write if stream and then we'll read the file here and then eventually we'll just close that file stream however if it didn't work out right we need to handle that as well
- so actually we might do that and like if I just write this code a bit differently we would basically return our string that we read so let's just say we had an std::string with our results and the we basically read that file over here and then we returned our result right that would be what it would be like if it were successful and then if it's not successful well what do we return right we could just return an empty string or we could return a string constructed like this which is the same exact result but the point is it's kind of hard for us to know if that file was open successfully or not because you can imagine if we have our data here and it's our rate file as string and we have our file path which is let us say data.txt if we have that here what how do we check if that was open successfully 
- well we basically have to do something like if data doesn't equal nothing right and then treated is that that's not nice at all what I would much prefer is some way of actually knowing if that data was read successfully or not because again maybe that file was there but is was empty that might be valid we need a way to know if it's not valid and so another solution might be to basically have this output boolean right which was you know a success let's just say right it's a boolean that we passed by reference is going to be our output variable 
- so I might call it our success and then if the stream was opened you know it will set success to true and we'll return that other wise we'll success well that success to false and we'll basically just return an empty string 
- so in that case instead of doing this we need to take in a boolean which is basically file open successfully and then we simply pass that into here right and then we can check that and that obviously reads a lot better than an empty string however it's still not particularly nice so that is where std::optional can help us a std::optional which we can include by just including the optional header file here is basically exactly what it sounds like it's optional as to whether or not the data is present so we changed this to be std::optional and then of course we still have our string as our type that is optional and you can see that is giving me an error here saying that namespace std has no remember optional as I mentioned earlier this is a C++17 feature so make sure that you go into your settings into your compiler settings switch your language version to C++ 17 and then now what we do is we still treat everything as it was before and somehow I removed the <fstream> include for some reason
- so we still do everything that we did before or I will get rid of this our success boolean we don't need that anymore and then instead of returning an empty string what we can do is return an empty as std::optional and you can do that just by having curly brackets like this and then if we want our string to be returned we simply return result and you can see it's really that simple and then if we scroll down into main here we  don't need this anymore we don't need this we change this to be an std::optional<std::string> right or optionally you could use order right that's also file and then what we do is we just check data dot has value and if has value is true that means that optional has been set and we can basically we'll just log out to the console file up read successfully and by the way just a quick not while I was editing this video this is future Cherno talking you can also just right if data because there's a bull operator so  instead of having to write data has value if data is also totally legit and I think it looks even cleaner and nicer anyway back to pasture no otherwise if it doesn't have value then we know that file was not read successfully so file could not be opened 
- okay so we have two different branches here based on whether or not that data was actually set on and of course to access it we have a bunch of ways to access it as well we can simply use data and an error and then we just have our string there we can also simply dereference a string so we'll say this is our string and it's just basically data dereference like that and we can access it as a normal string so it's much the same as a smart pointer in terms of if you want to actually access that data and in fact I  think there's also a data dot value which you can use 
- so let's test this out and see if that works what I'll do is just create a dummy file here I'll make sure that I'm in show all files and just in project I will create a new file and I'll just call it data.txt we might just throw in some data into here and then here it is in my solution Explorer what I'll do is try and load it so there's data text of course if I hit I'll just throw in a little C in yet here
- so that I'll console doesn't close immediately and we'll just run this program you can see the file was read successfully

```c++
#include <fstream>
#include <optional>
#include <iostream>

std::optional<std::string> ReadFileAsString(const std::string& filepath) {
	std::ifstream stream(filepath);
	if (stream) {
		std::string result;
		//read file
		stream.close();
		return result;
	}
	else {
		return {};
	}
}

int main() {
	std::optional<std::string> data = ReadFileAsString("data.txt");
	if (data.has_value()) {
		//std::string& string = *data;
		data.value();
		std::cout << "File read successfully\n";
	}
	else {
		std::cout << "File could bot be opened\n";
	}
	std::cin.get();
}
```

- now let's just say that file didn't exist I'll go ahead and delete it we'll hit F5 and you can see that it says file cannot be opened so easily now we're able to switch on whether or not that file was read successfully now there's one other really useful thing that we can do with this let's just say that we're trying to read the file and we don't really care too much about whether or not the file was read or not like us in we still care about it but it's not the end of the world if the file couldn't be opened because if the file couldn't be opened or if that particular section of the file wasn't set or wasn't read maybe we have a default value for it that's also quite common so what we can actually do is have std::string value and we can set this to data dot value or alright and what this will do is exactly what it sounds like if the if the data is actually present in that std::optional it will return to us that string if it's not it will return whatever value we pass in here so we can say you know not present 
- for example right this is really useful for if you're actually pausing files and trying to for example extract like variables or any kind of elements that have been set because quite often you might have something that is literally optional to have in the file and if a certain parameter was not set in the file you can go with a default value of what you specify here this is so useful because for example we could just simply have an int here count for example maybe we're trying to run a benchmark and in the file it says how many times are on the benchmark 
- now if this was present in the file we can extract that count right by just calling counts dot value or if it wasn't present we can go with the default value of a hundred and you can see how we can do that because this case this would come from the file

```c++
std::string value = data.value_or("Not present");

std::optional<int> count;
int c = count.value_or(0);
```



- so let's 

