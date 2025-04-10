---
title: 如何正确地将Unity导出的UWP工程的内容区域拓展到标题栏
date: 2023-06-22
updated: 2023-06-22
---

## 是什么工程
CX/C++工程

## 在哪里修改
在App.xaml.cpp

## 添加头文件

```C++
#include <Windows.UI.ViewManagement.h>
#include <Windows.ApplicationModel.Core.h>
#include <Windows.UI.ViewManagement.h>
```

## 在哪里添加代码

```C++
void App::InitializeUnity(String^ args)
{
    ApplicationView::GetForCurrentView()->SuppressSystemOverlays = true;
}

// 扩展内容区域到标题栏
auto coreTitleBar = Windows::ApplicationModel::Core::CoreApplication::GetCurrentView()->TitleBar;
coreTitleBar->ExtendViewIntoTitleBar = true;
```

 ## 最终全文

```C++
//
// App.xaml.cpp
// Implementation of the App class.
//

#include "pch.h"
#include "MainPage.xaml.h"
#include "UnityGenerated.h"
#include <Windows.UI.ViewManagement.h>
#include <Windows.ApplicationModel.Core.h>
#include <Windows.UI.ViewManagement.h>

using namespace majiang0;

using namespace Platform;
using namespace UnityPlayer;
using namespace Windows::ApplicationModel::Activation;
using namespace Windows::UI::ViewManagement;
using namespace Windows::UI::Xaml;
using namespace Windows::UI::Xaml::Controls;
using namespace Windows::UI::Xaml::Interop;


// The Blank Application template is documented at http://go.microsoft.com/fwlink/?LinkId=402347&clcid=0x409

// Hint that the discrete gpu should be enabled on optimus/enduro systems
// NVIDIA docs: http://developer.download.nvidia.com/devzone/devcenter/gamegraphics/files/OptimusRenderingPolicies.pdf
// AMD forum post: http://devgurus.amd.com/thread/169965
extern "C"
{
    __declspec(dllexport) DWORD NvOptimusEnablement = 0x00000001;
    __declspec(dllexport) int AmdPowerXpressRequestHighPerformance = 1;
}

/// <summary>
/// Initializes the singleton application object.  This is the first line of authored code
/// executed, and as such is the logical equivalent of main() or WinMain().
/// </summary>
App::App()
{
    InitializeComponent();
    SetupOrientation();
    m_AppCallbacks = ref new AppCallbacks();
}

/// <summary>
/// Invoked when the application is launched normally by the end user.  Other entry points
/// will be used such as when the application is launched to open a specific file.
/// </summary>
/// <param name="e">Details about the launch request and process.</param>
void App::OnLaunched(LaunchActivatedEventArgs^ e)
{
    m_SplashScreen = e->SplashScreen;
    InitializeUnity(e->Arguments);
}

/// <summary>
/// Invoked when application is launched through protocol.
/// Read more - http://msdn.microsoft.com/library/windows/apps/br224742
/// </summary>
/// <param name="args"></param>
void App::OnActivated(IActivatedEventArgs^ args)
{
    String^ appArgs = "";

    if (args->Kind == ActivationKind::Protocol)
    {
        auto eventArgs = safe_cast<ProtocolActivatedEventArgs^>(args);
        m_SplashScreen = eventArgs->SplashScreen;
        appArgs += String::Concat("Uri=", eventArgs->Uri->AbsoluteUri);
    }

    InitializeUnity(appArgs);
}

/// <summary>
/// Invoked when application is launched via file
/// Read more - http://msdn.microsoft.com/library/windows/apps/br224742
/// </summary>
/// <param name="args"></param>
void App::OnFileActivated(FileActivatedEventArgs^ args)
{
    String^ appArgs = "File=";

    m_SplashScreen = args->SplashScreen;

    bool firstFileAdded = false;

    for (auto file : args->Files)
    {
        if (firstFileAdded)
        {
            appArgs += ";";
        }
        else
        {
            firstFileAdded = true;
        }

        appArgs += file->Path;
    }

    InitializeUnity(appArgs);
}

void App::InitializeUnity(String^ args)
{
    ApplicationView::GetForCurrentView()->SuppressSystemOverlays = true;

    // 扩展内容区域到标题栏
    auto coreTitleBar = Windows::ApplicationModel::Core::CoreApplication::GetCurrentView()->TitleBar;
    coreTitleBar->ExtendViewIntoTitleBar = true;

    m_AppCallbacks->SetAppArguments(args);
    auto rootFrame = safe_cast<Frame^>(Window::Current->Content);

    // Do not repeat app initialization when the Window already has content,
    // just ensure that the window is active
    if (rootFrame == nullptr && !m_AppCallbacks->IsInitialized())
    {
        rootFrame = ref new Frame();
        Window::Current->Content = rootFrame;
#if !UNITY_HOLOGRAPHIC
        Window::Current->Activate();
#endif

        rootFrame->Navigate(TypeName(MainPage::typeid ));
    }

    Window::Current->Activate();
}

void App::SetupOrientation()
{
    Unity::SetupDisplay();
}
```
