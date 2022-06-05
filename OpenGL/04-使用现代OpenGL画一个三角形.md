- 现代OpenGL比传统OpenGL更具有可编程性，它的拓展性更好，也更强大，能用它做许多事情，结果就是在屏幕上绘制一个三角形之前，实际上需要做很多设置 
- 第二小节使用传统OpenGL,只用了glBegin()和glEnd()在屏幕上绘制一个三角形，传统OpenGL做起来很容易，并且不需要什么设置。然而，使用现代OpenGL去画一个三角形，需要做一些事情。首先，我们需要能够创建一个顶点缓冲区，然后还要创建一个着色器(shader)。
- 顶点缓冲区(vertex buffer)它基本上就是去掉vertex这个单词，从它的名字去掉vertex这个单词，它只是一个缓冲区，一个内存缓冲区，一个内存字节数组，那是它的本质，只是一个缓冲区。但顶点缓冲区跟C++中像字符数组的内存缓冲区不太一样，区别在于顶点缓冲区是OpenGL中的内存缓冲区，这意味着它实际上在显卡上，在我们的VRAM(显存)中，也就是Video RAM。
- 使用现代OpenGL绘制一个三角形的基本思路就是，定义一些数据来表示三角形，把它放入显卡的VRAM中，然后还需要发出DrawCall指令，这是一个绘制指令。实际上，我们还要告诉显卡如何读取和解释这些数据，以及如何把它放到我们的屏幕上
- 一旦从CPU发出了DrawCall指令，我们就告诉显卡， 它要做什么，所以需要对显卡编程，这就是着色器(shader).着色器是一个运行在显卡上的程序，是一堆我们可以编写的在显卡上运行的代码，它可以在显卡上以一种非常特殊又非常强大的方式运行
- OpenGL具体的操作就是一个状态机，也就是不需要把它看成对象或类似的东西，所做的是设置一系列的状态，然后告诉它给我绘制个与上下文相关的三角形，所以换句话说，我想让你选择这个缓冲区，用这个着色器，然后给我绘制个三角形
- 根据选择的缓冲区和着色器，决定绘制什么样的三角形，绘制在哪里等等，这就是OpenGL的原理，它是一个状态机
- 定义顶点缓冲区unsigned int buffer; glGenBuffers(1, &buffer); 第一个参数是定义多少个缓冲区，第二个参数需要一个无符号整型指针，glGenBuffers()这个函数返回void,所以函数不返回生成的缓冲区id, 我们需要给它提供一个整数指针，然后函数把id写入这个整形指针。

- 上下文有自己的流，缓冲区有自己的流，但在OpenGL中生成的所有东西都会被分配一个唯一的标识符，它只是一个整数，比如0，1，2。不管它是顶点缓冲区，顶点数组，纹理(texture)，着色器(shader)或者其他任何东西
- 现在我们有了一个id(buffer)，一旦创建缓冲区后，我们就要选择那个缓冲区，并且选择(selecting)在OpenGL中称为绑定(binding)。glBindBuffer(GL_ARRAY_BUFFER， buffer),第一个参数是目标GL_ARRAY_BUFFER，也就是说这只是一个数组.第二个参数就是我们要绑定的缓冲区
- 下一步是指定数据。在创建缓冲区时，指定它的大小，然后直接给出数据。glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), positions, GL_STATIC_DRAW);第二个参数是指缓冲区的大小, 第三个参数数据指针 ,最后一个参数GLenum usage参数

```cpp
float positions[6] = {
  	-0.5f, -0.5f,
     0.0f,  0.5f,
     0.5f, -0.5f
};
unsigned int buffer;
glGenBuffers(1, &buffer);
glBindBuffer(GL_ARRAY_BUFFER, buffer);
glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), positions, GL_STATIC_DRAW);
```

```cpp
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <iostream>

int main(void)
{
    GLFWwindow* window;

    /* Initialize the library */
    if (!glfwInit())
        return -1;

    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    if (glewInit() != GLEW_OK) {
        std::cout << "Error" << std::endl;
    }

    std::cout << glGetString(GL_VERSION) << std::endl;

    //定义OpenGL顶点缓冲区的数据 
    float positions[6] = {
    -0.5f, -0.5f,
     0.0f,  0.5f,
     0.5f, -0.5f
    };
    unsigned int buffer;
    glGenBuffers(1, &buffer);
    glBindBuffer(GL_ARRAY_BUFFER, buffer);
    glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), positions, GL_STATIC_DRAW);

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Render here */
        glClear(GL_COLOR_BUFFER_BIT);
        
        glDrawArrays(GL_TRIANGLES, 0, 3);

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```

