# govcl
### golang binding delphi vcl framework

### 项目说明  

目前项目只支持Windows(支持Win32、Win64)，但代码中写了借助于Crossvcl实现支持Linux与Mac OS的，无奈暂时Crossvcl在dylib中直接创建会报错，所以跨平台这块暂时停止了开发。    


> 本仓库并不会提交实体代码(就目前情况至少不会)，代码现存于OSC私有项目中，目前还未完成整个项目，有些还需要修改的，后续视情况来开源吧。  
 

### 实例类说明

** 按照Delphi中的Application、 Screen、 Mouse、Clipboard四个类实例是可以直接访问的，不需要释放  
其实组件带有Onwer参数的一般指定TFrom的就好了，这样就不需要手动释放，反之Owner填   
写nil则需要手动调用Free，就像其它非组件类的。 **   

> 现支持组件和非组件类列表：  
>
TApplication    
TForm    
TButton    
TEdit    
TMainMenu    
TPopupMenu    
TMemo    
TCheckBox    
TRadioButton    
TGroupBox    
TLabel    
TListBox    
TComboBox    
TPanel    
TImage    
TLinkLabel    
TSpeedButton   
TSplitter    
TRadioGroup    
TStaticText    
TColorBox    
TColorListBox    
TTrayIcon    
TBalloonHint    
TCategoryPanelGroup    
TOpenDialog    
TSaveDialog    
TColorDialog    
TFontDialog    
TPrintDialog    
TOpenPictureDialog    
TSavePictureDialog    
TSaveTextFileDialog    
TOpenTextFileDialog    
TRichEdit    
TTrackBar    
TImageList    
TUpDown    
TProgressBar    
THotKey    
TDateTimePicker    
TMonthCalendar    
TListView    
TTreeView    
TStatusBar    
TToolBar    
TPageControl  
TTabSheet    
TControl 
TActionList  
TToolButton     
> 
TIcon    
TBitmap    
TMemoryStream    
TFont    
TStrings    
TStringList    
TBrush    
TPen    
TMenuItem    
TListGroups    
TPicture    
TListColumns    
TListItems    
TTreeNodes    
TListItem    
TTreeNode      
TScreen    
TMouse    
TListGroup    
TListColumn    
TCollectionItem    
TStatusPanels    
TStatusPanel    
TCanvas    
TObject    
TPngImage    
TJPEGImage    
TGIFImage    
TGIFFrame    
TIniFile  
TRegistry  
TClipboard  



** 文件名后面带有def的为手动编写 **   


### 截图  

[截图1](https://raw.githubusercontent.com/ying32/govcl/master/Screenshot/1.png)   
[截图2](https://raw.githubusercontent.com/ying32/govcl/master/Screenshot/2.png)    


#### 代码示例  


```golang
// govcl project main.go
package main

// windows下编译标识
// go.exe build -i -ldflags="-H windowsgui"

import (
	"fmt"

	// 这里导入项目路径
)

var (
	mainForm *vcl.TForm
	trayicon *vcl.TTrayIcon
)

func main() {

	// 异常捕获
	defer func() {
		err := recover()
		if err != nil {
			fmt.Println("Exception: ", err)
			vcl.ShowMessage(err.(error).Error())
		}
	}()

	//rtl.SetReportMemoryLeaksOnShutdown(true)
	fmt.Println("main")
	icon := vcl.NewIcon()
	//icon.LoadFromFile("0.ico")
	icon.LoadFromResourceID(rtl.MainInstance(), 3)
	defer icon.Free()
	vcl.Application.Initialize()
	vcl.Application.SetIcon(icon)
	vcl.Application.SetTitle("Hello World!")
	vcl.Application.SetMainFormOnTaskBar(true)
	mainForm = vcl.Application.CreateForm()
	mainForm.SetWidth(600)
	mainForm.SetHeight(400)
	mainForm.SetOnClose(func(Sender vcl.IObject, Action uintptr) {
		fmt.Println("close")
	})

	mainForm.SetOnCloseQuery(func(Sender vcl.IObject, CanClose uintptr) {
		//		rtl.FormSetCanClose(CanClose, false)
		fmt.Println("OnCloseQuery")
	})

	mainForm.SetCaption(vcl.Application.Title())
	mainForm.EnabledMaximize(false)
	mainForm.SetDoubleBuffered(true)

	trayicon = vcl.NewTrayIcon(mainForm)
	trayicon.SetIcon(icon)
	trayicon.SetHint(mainForm.Caption())
	trayicon.SetVisible(true)
	trayicon.SetOnClick(func(vcl.IObject) {
		trayicon.SetBalloonTitle("test")
		trayicon.SetBalloonTimeout(10000)
		trayicon.SetBalloonHint("我是提示正文啦")
		trayicon.ShowBalloonHint()
		fmt.Println("TrayIcon Click.")
	})

	// img
	img := vcl.NewImage(mainForm)
	img.SetBounds(132, 30, 156, 97)
	img.SetParent(mainForm)
	img.Picture().LoadFromFile(".\\imgs\\1.jpg")
	//img.SetStretch(true)
	img.SetProportional(true)

	// linklabel
	linklbl := vcl.NewLinkLabel(mainForm)
	linklbl.SetAlign(api.AlBottom)
	linklbl.SetCaption("<a href=\"http://www.baidu.com\">测试链接</a>")
	linklbl.SetParent(mainForm)
	linklbl.SetOnLinkClick(func(sender vcl.IObject, link string, linktype int32) {
		fmt.Println("link label: ", link, ", type: ", linktype)
	})

	// menu
	mainMenu := vcl.NewMainMenu(mainForm)
	item := vcl.NewMenuItem(mainForm)
	item.SetCaption("File(&F)")
	mainMenu.Items().Add(item)

	item2 := vcl.NewMenuItem(mainForm)
	item2.SetCaption("MemoryStreamTest")
	item2.SetOnClick(func(vcl.IObject) {
		mem := vcl.NewMemoryStream()
		defer mem.Free()
		s := api.GoStrToDStr("ffff中f")
		mem.Write(s, int32(api.DStrLen(s)*2))
		mem.SaveToFile("test.txt")
	})
	item.Add(item2)

	item2 = vcl.NewMenuItem(mainForm)
	item2.SetCaption("Exit(&E)")
	item2.SetShortCutFromString("Ctrl+Q")
	item2.SetOnClick(func(vcl.IObject) {
		mainForm.Close()
	})
	item.Add(item2)

	//	mainForm.EnabledMinimize(false)
	//	mainForm.EnabledSystemMenu(false)

	button := vcl.NewButton(mainForm)

	button.SetCaption("消息")
	button.SetParent(mainForm)
	button.SetOnClick(func(vcl.IObject) {
		fmt.Println("button click")
		vcl.ShowMessage("这是一个消息")
		vcl.Application.MessageBox("Hello!", "Message", win.MB_YESNO+win.MB_ICONINFORMATION)
	})
	button.SetLeft(50)
	button.SetTop(50)
	button.SetAlign(api.AlRight)

	edit := vcl.NewEdit(mainForm)
	edit.SetParent(mainForm)
	edit.SetLeft(1)
	edit.SetTop(30)
	edit.SetTextHint("测试")
	edit.SetOnChange(func(vcl.IObject) {
		fmt.Println("edit OnChange")
	})

	button2 := vcl.NewButton(mainForm)
	button2.SetParent(mainForm)
	button2.SetCaption("a")
	button2.SetWidth(100)
	button2.SetHeight(28)
	button2.SetOnClick(func(vcl.IObject) {
		fmt.Println("button2 click")

		edit.SetText("Hello!")
		fmt.Println("ScreenWidth:", vcl.Screen.Width(), ", ScreenHeight:", vcl.Screen.Height())
	})
	button2.SetAlign(api.AlTop)

	chk := vcl.NewCheckBox(mainForm)
	chk.SetParent(mainForm)
	chk.SetChecked(true)
	chk.SetCaption("测试")
	chk.SetLeft(1)
	chk.SetTop(60)
	chk.SetOnClick(func(vcl.IObject) {
		fmt.Println("chk.Checked=", chk.Checked())
	})

	combo := vcl.NewComboBox(mainForm)
	combo.SetAlign(api.AlBottom)
	combo.SetParent(mainForm)
	combo.SetText("ffff")
	combo.Items().Add("1")
	combo.Items().Add("2")
	combo.SetItemIndex(0)
	combo.SetOnChange(func(vcl.IObject) {
		if combo.ItemIndex() != -1 {
			fmt.Println("combo Change: ", combo.Items().Strings(combo.ItemIndex()))
		}

	})

	page := vcl.NewPageControl(mainForm)
	page.SetParent(mainForm)
	page.SetAlign(api.AlBottom)
	sheet := vcl.NewTabSheet(mainForm)
	sheet.SetPageControl(page)
	sheet.SetCaption("第一页")

	// 需要先将TabSheet设置了父窗口，TListView才可用，不然就会报错
	lv1 := vcl.NewListView(mainForm)
	lv1.SetAlign(api.AlClient)
	lv1.SetParent(sheet)

	lv1.SetViewStyle(api.VsReport)
	lv1.SetRowSelect(true)
	lv1.SetReadOnly(true)
	lv1.SetGridLines(true)
	col := lv1.Columns().Add()
	col.SetCaption("序号")
	col.SetWidth(100)
	col = lv1.Columns().Add()
	col.SetCaption("名称")
	col.SetWidth(200)
	lv1.SetOnClick(func(vcl.IObject) {
		if lv1.ItemIndex() != -1 {
			fmt.Println(lv1.Items().Item(lv1.ItemIndex()).Caption())
		}
	})

	lv1.Items().BeginUpdate()
	for i := 1; i <= 50; i++ {
		lstitem := lv1.Items().Add()
		lstitem.SetCaption(fmt.Sprintf("%d", i))
		lstitem.SubItems().Add(fmt.Sprintf("第%d", i))
	}
	lv1.Items().EndUpdate()

	sheet = vcl.NewTabSheet(mainForm)
	sheet.SetCaption("第二页")
	sheet.SetPageControl(page)

	tv1 := vcl.NewTreeView(mainForm)
	tv1.SetParent(sheet)
	tv1.SetAlign(api.AlClient)
	tv1.Items().AddChild(nil, "首个")

	mainForm.ScreenCenter()

	vcl.Application.Run()
}


```  