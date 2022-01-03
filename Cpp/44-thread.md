```c++
#include <iostream>
#include <thread>
static bool finished = false;
void doWork() {
	using namespace std::literals::chrono_literals;
	while (!finished) {
		std::cout << "Working...\n";
		std::this_thread::sleep_for(1s);
	}
}
int main() {
    //开启一个线程
	std::thread worker(doWork);
	std::cin.get();
	finished = true;
	worker.join();
	
	std::cin.get();
}
```

