Внедрение Dependency Injection (DI) в Windows Forms проекте на C# позволяет разделить ответственность и улучшить тестируемость и масштабируемость кода. Несмотря на то, что DI чаще используется в веб-приложениях, его можно легко применить в WinForms с помощью сторонних DI-контейнеров, таких как **Microsoft.Extensions.DependencyInjection** или **Autofac**.

### Как внедрить DI в проект Windows Forms:

1. **Установите пакет DI-контейнера:** Для примера будем использовать Microsoft.Extensions.DependencyInjection. Установите пакет через NuGet:
    
    `Install-Package Microsoft.Extensions.DependencyInjection`
    
2. **Создайте контейнер для DI:** Настройте зависимости приложения в одном месте, например, в `Program.cs`.
    
3. **Пример настройки контейнера и внедрения DI:**
    

```cs
using System;
using Microsoft.Extensions.DependencyInjection;
using System.Windows.Forms;

namespace WinFormsDIExample
{
    internal static class Program
    {
        // Главный контейнер DI
        private static ServiceProvider _serviceProvider;

        [STAThread]
        static void Main()
        {
            // Настройка DI
            var services = new ServiceCollection();
            ConfigureServices(services);

            _serviceProvider = services.BuildServiceProvider();

            // Запуск приложения
            Application.SetHighDpiMode(HighDpiMode.SystemAware);
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);

            var mainForm = _serviceProvider.GetRequiredService<MainForm>();
            Application.Run(mainForm);
        }

        private static void ConfigureServices(IServiceCollection services)
        {
            // Регистрируем зависимости
            services.AddSingleton<IMyService, MyService>();
            services.AddTransient<MainForm>(); // Регистрируем форму
        }
    }

    // Пример интерфейса
    public interface IMyService
    {
        string GetData();
    }

    // Реализация интерфейса
    public class MyService : IMyService
    {
        public string GetData()
        {
            return "Hello from MyService!";
        }
    }

    // Главная форма
    public class MainForm : Form
    {
        private readonly IMyService _myService;

        public MainForm(IMyService myService)
        {
            _myService = myService;

            var button = new Button { Text = "Click me" };
            button.Click += Button_Click;
            Controls.Add(button);
        }

        private void Button_Click(object sender, EventArgs e)
        {
            MessageBox.Show(_myService.GetData());
        }
    }
}

```

### Что здесь происходит:

1. **Настройка DI-контейнера:** Все зависимости регистрируются в методе `ConfigureServices`. Формы и сервисы регистрируются как `Transient`, `Scoped` или `Singleton`, в зависимости от необходимости.
    
2. **Разделение ответственности:** Логика (`IMyService`) вынесена в отдельный сервис, который внедряется в `MainForm`.
    
3. **Получение зависимостей:** `MainForm` получает свои зависимости через конструктор благодаря DI.