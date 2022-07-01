- Windows的图形接口，DirectX 或者 Direct3D

- GLFW能做的就是在一个头文件中提供了OpenGL接口规范，各种函数声明，符号声明和常量等等。这种背后的文件，这种场景的C++文件，被认为是安全的文件。这个库的实际实现，就是进入EDI,在你使用的显卡驱动签名中查找对应的动态链接文件，然后载入所有这些函数指针。不要认为GLFW库实现了一些函数或其他东西， 它只是访问这些函数，那些函数早就以二进制的形式存在在电脑上了。我们使用这个库只是为我们做了一些事情，比如GLEW, OpenGL extension wrangler(OpenGL扩展管理)。也有一个GLAD库，这是OpenGL的一个比较特殊的扩展

- [glew]:glew.sourceforge.net

- 不能从GLEW中直接调用OpenGL函数，直到调用了glewInit()之后。需要创建一个渲染OpenGL的上下文，在你调用glewInit()之前(在glfwMakeContextCurrent(window);方法之后再调用glewInit()方法)

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

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Render here */
        glClear(GL_COLOR_BUFFER_BIT);
        
        glBegin(GL_TRIANGLES);
        glVertex2f(-0.5f, -0.5f);
        glVertex2f(0.0f, 0.5f);
        glVertex2f(0.5f, -0.5f);
        glEnd();

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```

