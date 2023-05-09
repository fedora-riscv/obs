# Bug 1
在编译完ROS的Fedora RISC-V上运行roscore，随后会卡住，CPU占用率100%
在经过排查后发现是 https://github.com/ros/ros_comm/blob/030e132884d613e49a576d4339f0b8ec6f75d2d8/tools/rosgraph/src/rosgraph/roslogging.py#L64 该循环的问题

```python
        file_name, lineno, func_name = super(RospyLogger, self).findCaller(*args, **kwargs)[:3]
        file_name = os.path.normcase(file_name)

        f = inspect.currentframe()
        if f is not None:
            f = f.f_back
        while hasattr(f, "f_code"):
            # Search for the right frame using the data already found by parent class.
            co = f.f_code
            filename = os.path.normcase(co.co_filename)
            if filename == file_name and f.f_lineno == lineno and co.co_name == func_name:
                break
            if f.f_back:
                f = f.f_back
```
该段代码应该只是递归调用栈寻找对应栈，在x86_64环境下应该循环了两到三次即可找到，但Fedora RISC-V环境下将不停向上找，知道最顶端调用链，而此时`f == f.f_back`，导致出现死循环
目前尚且不清楚是inspect包的问题还是riscv64架构使然
