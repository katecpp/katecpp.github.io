---
layout: post
title:  "How to access Outlook e-mail with C++"
permalink: outlook-with-cpp/
date:   2016-07-18
category: cpp
---
There is an example in Qt docs of [how to access Outlook e-mail with C++ and MSOUTL](http://doc.qt.io/qt-5/activeqt-activeqt-qutlook-example.html), but for me the described scenario didn't work as expected and I needed to modify some steps. Here is what I've done to make it work. I used Windows 7 and Outlook 14.

#### Create MSOUTL

Instead of creating MSOUTL sources from .pro file, I've created it manually and copy-pasted the resulting files into the repository. It was necessary because later I had to modify these files. So, I skipped the TYPELIBS line in .pro, and I've done it manually from cmd (adjust your paths to Qt and Microsoft Office):

{% highlight shell %}
D:\Qt\5.5\mingw492_32\bin\dumpcpp.exe C:\Program Files (x86)\Microsoft Office\Office14\MSOUTL.OLB -o MSOUTL
{% endhighlight %}
Files MSOUTL.h and MSOUTL.cpp are created. I included them to project like normal source files.

#### Fix "Use of enum 'MsoFeatureInstall' without previous declaration..."

With these files included, I've tried to compile basic application of *Outlook::Application*. It gave me errors like:

> use of enum 'MsoFeatureInstall' without previous declaration
> enum MsoFeatureInstall;

I've learned that this is [an already reported bug](http://www.qtcentre.org/archive/index.php/t-8811.html).
I've applied suggested fix to MSOUTL.h:

{% highlight cpp %}

// Referenced namespace
namespace Office {
    // Fix -->
    enum MsoFeatureInstall {
    msoFeatureInstallNone = 0,
    msoFeatureInstallOnDemand = 1,
    msoFeatureinstallOnDemandWithUI = 2
    };
    // <-- Fix
    class Assistant;
    class COMAddIns;
    class LanguageSettings;
    class AnswerWizard;
    enum MsoFeatureInstall;
    class IAssistance;
    class PickerDialog;
    class ContactCard;
    class CommandBars;
    class CommandBar;
}
{% endhighlight %}

It helped and my code compiled.


#### Proper usage

After successful setup reading e-mails is easy. Following code reads subjects and bodies of e-mails from Inbox:

{% highlight cpp %}
Outlook::Application outlook;
if (!outlook.isNull())
{
    Outlook::NameSpace session(outlook.Session());
    session.Logon();
    Outlook::MAPIFolder *folder = session.GetDefaultFolder(Outlook::olFolderInbox);

    Outlook::Items* mails = new Outlook::Items(folder->Items());
    mails->Sort("ReceivedTime");

    // Indexing starts from 1
    for (int i = 1; i < 10; i++)
    {
        Outlook::MailItem mail(mails->Item(i));
        QString s = mail.Subject(); // do something with subject
        QString b = mail.Body(); // do something with body
    }
}
{% endhighlight %}

The minimal app for reading e-mails is on my [Github/MailReader](https://github.com/katecpp/MailReader). I didn't include MSOUTL files there as I'm not sure about their licence and in fact the library can be different for various Outlook versions.


#### Troubleshooting

If you try to implement very simple usage of MSOUTL and you have errors like:

> QAxBase::setControl: requested control {0006f03a-0000-0000-c000-000000000046} could not be instantiated

then use solution suggested [in this topic](http://www.qtcentre.org/threads/24583-Outlook-problem). I've had this problem when I tried to create *Outlook::Application* object directly in main() function. When I moved it to class and wrapped it with a Widget, the problem disappeared.
