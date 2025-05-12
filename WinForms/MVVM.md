
**ProcessListItem.cs**
```cs
using System.Diagnostics;

namespace FirstWPF
{
    public class ProcessListItem
    {
        public int Id => Process.Id;
        public string ProcessName => Process.ProcessName;
        public bool KeepAlive { get; set; }
        public Process Process { get; }
        public string FileName { get; }
        public string Arguments { get; }

        public ProcessListItem(Process process)
        {
            Process = process;
            FileName = process.StartInfo.FileName;
            Arguments = process.StartInfo.Arguments;
        }
    }
}
```

**MainWindow.xaml.cs**
```cs
using System;
using System.Diagnostics;
using System.Linq;
using System.Windows;
using System.Windows.Controls;

namespace FirstWPF
{
    public partial class MainWindow
    {
        public MainWindow()
        {
            InitializeComponent();
            Priorities.ItemsSource = Enum.GetValues(typeof(ProcessPriorityClass)).Cast<ProcessPriorityClass>();
            Priorities.SelectedIndex = 0;
        }

        private void TasksList_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            var listBox = (ListBox)sender;

            if (listBox.SelectedItems.Count > 0)
            {
                var viewModel = (ViewModel)DataContext;
                viewModel.SelectedProcess = ((ProcessListItem)listBox.SelectedItems[0]).Process;
            }
        }

        private void RefreshButton_OnClick(object sender, RoutedEventArgs e)
        {
            var viewModel = (ViewModel)DataContext;
            viewModel.UpdateProcesses(sender, e);
        }

        private void ChangePriority_OnClick(object sender, RoutedEventArgs e)
        {
            var priority = (ProcessPriorityClass)Priorities.SelectionBoxItem;
            var viewModel = (ViewModel)DataContext;
            viewModel.ChangePriority(priority);
        }

        private void KillProcess_OnClick(object sender, RoutedEventArgs e)
        {
            var viewModel = (ViewModel)DataContext;
            viewModel.KillSelectedProcess();
        }

        private void ToggleButton_OnChecked(object sender, RoutedEventArgs e)
        {
            var checkBox = (CheckBox)sender;
            var processListItem = (ProcessListItem)checkBox.DataContext;
            processListItem.KeepAlive = true;
        }

        private void ToggleButton_OnUnchecked(object sender, RoutedEventArgs e)
        {
            var checkBox = (CheckBox)sender;
            var processListItem = (ProcessListItem)checkBox.DataContext;
            processListItem.KeepAlive = false;
        }
    }
}
```


**ViewModel.cs**
```cs
using System;
using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Diagnostics;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Windows.Threading;
using TaskManager.Annotations;

namespace FirstWPF
{
    public class ViewModel : INotifyPropertyChanged
    {
        public ViewModel()
        {
            var timer = new DispatcherTimer { Interval = TimeSpan.FromSeconds(2) };
            timer.Tick += UpdateProcesses;
            timer.Start();
        }

        private Process _selectedProcess;
        public Process SelectedProcess
        {
            get => _selectedProcess;
            set
            {
                _selectedProcess = value;
                OnPropertyChanged();
            }
        }

        public ObservableCollection<ProcessListItem> Processes { get; } = new ObservableCollection<ProcessListItem>();

        public void UpdateProcesses(object sender, EventArgs e)
        {
            var currentIds = Processes.Select(p => p.Id).ToList();

            foreach (var p in Process.GetProcesses())
            {
                if (!currentIds.Remove(p.Id)) // it's a new process id
                {
                    Processes.Add(new ProcessListItem(p));
                }
            }

            foreach (var id in currentIds) // these do not exist any more
            {
                var process = Processes.First(p => p.Id == id);
                if (process.KeepAlive)
                {
                    Process.Start(process.ProcessName, process.Arguments);
                }
                Processes.Remove(process);
            }
        }

        public void ChangePriority(ProcessPriorityClass priority)
        {
            SelectedProcess.PriorityClass = priority;
        }

        public void KillSelectedProcess()
        {
            SelectedProcess.Kill();
        }

        public event PropertyChangedEventHandler PropertyChanged;
        [NotifyPropertyChangedInvocator]
        private void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
```