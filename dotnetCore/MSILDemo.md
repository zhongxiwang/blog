
 使用msil代码构造函数
```c++{.line-numbers}


        public static void Test(Menum name)
        {
            int n = 3;
            Console.WriteLine(name.Name + ":" + n);
            IntPtr res= GCHandle.ToIntPtr(GCHandle.Alloc(name));
            long number= res.ToInt64();
            IntPtr address = new IntPtr(number);
            var m= GCHandle.FromIntPtr(address).Target as Menum;
            Console.WriteLine(m.Name);
        }
        //构造部分
            Type[] Margs = new Type[] { typeof(Menum) };
            DynamicMethod dm = new DynamicMethod("TestMethod",null, Margs, typeof(Program).Module);
            MethodInfo TestMe = typeof(Program).GetMethod("Test", TestPar);
            var codes= dm.GetILGenerator();
            codes.Emit(OpCodes.Ldarg_0);
            codes.EmitCall(OpCodes.Call, writeString, null);
            codes.Emit(OpCodes.Ret);
            Action<Menum> ac =(Action<Menum>) dm.CreateDelegate(typeof(Action<Menum>));
            ac(tmp);
```

示例二：
```c++{.line-numbers}
private delegate int HelloInvoker(Menum msg, int ret);



DynamicMethod hello = new DynamicMethod("Hello",
                typeof(int),
                helloArgs,
                typeof(Program).Module);

            Type[] writeStringArgs = { typeof(string) };
            Type[] TestPar = { typeof(Menum) };
            MethodInfo writeString = typeof(Program).GetMethod("Test", TestPar);
                //typeof(Console).GetMethod("WriteLine", writeStringArgs);
            

            ILGenerator il = hello.GetILGenerator();

            il.Emit(OpCodes.Ldarg_0);

            il.EmitCall(OpCodes.Call, writeString, null);

            il.Emit(OpCodes.Ldarg_1);
            il.Emit(OpCodes.Ret);

            HelloInvoker hi =
                (HelloInvoker)hello.CreateDelegate(typeof(HelloInvoker));

            // Use the delegate to execute the dynamic method. Save and
            // print the return value.

            var tmp = new Menum
            {
                Name = "锐雯",
                LicenseNumber = 122,
                Owner = 123
            };
            int retval = hi(tmp, 42);


```