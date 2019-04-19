+++
title = "Running as Windows Service"
description = ""
weight = 1
alwaysopen = true
+++

Every generated project has the required code to run as a windows service.

In Program.cs remove the line CreateWebHostBuilderForServiceIDE(args).Build().Run();

Uncomment the commented out code.

Make sure you're targeting windows in the csproj. 

When running in the IDE the project will run from the console like normal. With this code uncommented when running from the command line the default
is to be a windows service. You can still run the executable as a console app by passing --console on the command line. Otherwise
you can use SC create line normal to create your service. 


```
  public class Program
    {
		public static void Main(string[] args)
        {
            CreateWebHostBuilderForServiceIDE(args).Build().Run();

			/* uncomment to support hosting as a windows service 
			bool isService = !(Debugger.IsAttached || args.Contains("--console"));

            if (isService)
            {
                CreateWebHostBuilderForService(args).Build().RunAsCustomService();
            }
            else
            {
                CreateWebHostBuilderForServiceIDE(args).Build().Run();
            }
			*/
        }

        public static IWebHostBuilder CreateWebHostBuilderForServiceIDE(string[] args)
        {
            return WebHost.CreateDefaultBuilder(args)
                     .UseStartup<Startup>()
                     .UseKestrel()
                     .UseUrls("http://0.0.0.0:8000");
        }

        public static IWebHostBuilder CreateWebHostBuilderForService(string[] args)
        {
            string pathToExe = Process.GetCurrentProcess().MainModule.FileName;
            string pathToContentRoot = Path.GetDirectoryName(pathToExe);

            return WebHost.CreateDefaultBuilder(args)
                .ConfigureAppConfiguration((context, config) =>
                {
                })
                .UseContentRoot(pathToContentRoot)
                .UseStartup<Startup>()
                .UseKestrel()
                .UseUrls("http://0.0.0.0:8000");
        }
    }
```