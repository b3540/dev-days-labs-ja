# Xamarin Dev Days ハンズオン手順書

これは、Xamarin.Forms と Microsoft Azure を使った簡単なアプリを作るハンズオンです。

Microsoft 本社の Xamarin チームが作った、詳細なハンズオン手順書『[Xamrin Dev Days Hands On Lab](https://github.com/xamarin/dev-days-labs/tree/master/HandsOnLab)』の日本語訳版をここに載せます。


## 今回 何を作るの？

| 項目                       | 値                                                            |
|----------------------------|---------------------------------------------------------------|
| どんなアプリ？             | Xamarin Dev Days の本社スピーカーと、その人の詳細を表示するアプリ |
| どんなアプリ？(技術的視点)  | Microsoft Azure に接続された Xamarin.Forms で作るアプリ       |


## 開発環境

Windows でも Mac でも良いです。

|OS|OS のバージョン|要インストール済|
|----|----|----|
|Windows|Windows 10|.NET モバイル開発ワークロードをインストールした Visual Studio 2017|
|Mac OS X|10.11 ("El Capitan") 以降 |Visual Studio for Mac と 最新の Xcode |

# さぁ手を動かそう！

【重要】      
以下の本家のレポジトリをクローンまたはダウンロードしてローカルに保存し、開いておいてください。（この日本語リポジトリではありません）

[https://github.com/xamarin/dev-days-labs](https://github.com/xamarin/dev-days-labs)

![クローン先](image/clone.png)

## 手順 1 ：ソリューションファイルを開く

[`dev-days-labs/HandsOnLab/Start/` ディレクトリ](https://github.com/xamarin/dev-days-labs/tree/master/HandsOnLab/Start) の中にある「`DevDaysSpeakers.sln`」を開いてください。


ソリューションタブを見ると、4つのプロジェクトで構成されているのが分かります。

|#|プロジェクト名|概要|実行環境|
|----|----|----|----|
|1 | `DevDaysSpeakers`|共通コード部分 (model、view、view model など) が全部入った Shared Project。| |
|2 | `DevDaysSpeakers.Droid`|Android アプリケーション| |
|3 | `DevDaysSpeakers.iOS`|iOS アプリケーション| ビルドホストの macOS が必要|
|4 | `DevDaysSpeakers.UWP`|Windows 10 UWP アプリケーション|Windows 10 上の Visual Studio 2015/2017 が必要|

![DevDaysSpeakers.sln](./image/Solution001.png)

共通部分である `DevDaysSpeakers` プロジェクトの中に、空白の XAMLページ ([View/DetailsPage.xaml](https://github.com/xamarin/dev-days-labs/blob/master/HandsOnLab/Start/DevDaysSpeakers/DevDaysSpeakers/View/DetailsPage.xaml)など) がありますが、これはこのハンズオンの中で使うことになるものです。

![](https://camo.githubusercontent.com/4c30ee2264e05b54aae37cf029bcc56fd62e05f9/68747470733a2f2f626c6f672e78616d6172696e2e636f6d2f77702d636f6e74656e742f75706c6f6164732f323031342f30362f58616d6172696e466f726d73312e706e67)

## 手順 2 : NuGet Restore

すべてのプロジェクトにおいて、必要な NuGet パッケージはすべてインストール済みとなっています。なので、このハンズオンでは、新たにパッケージを追加でインストールする必要はありません。

まず最初に私たちがしなければならないことは、インターネットから、すべての NuGet パッケージをリストアすることです。

> Visual Studio for Mac では自動でリストアされます。

どうやるかというと、ソリューションタブの中の『ソリューション』を
右クリックして、『`Restore NuGet packages`』をクリックします。

![Restore NuGet packages](./image/Solution002.png)

ということで、アプリを「実行」してみましょう！    
IDEの「▶」(再生ボタンみたいな)を押して実行です

![完成形](./image/start_app.png)


## 手順 3 : Model を いじる

スピーカーの情報を取ってこれるようにするため、 Speaker モデルを作りましょう。

1. `DevDaysSpeakers (共通部分)` プロジェクトの中の `DevDaysSpeakers/Model/Speaker.cs` ファイルを開きます。
2. 【コピペ】 Speaker クラスの中に、以下のプロパティを追加してください。

```csharp
public string Id { get; set; }
public string Name { get; set; }
public string Description { get; set; }
public string Website { get; set; }
public string Title { get; set; }
public string Avatar { get; set; }
```

## 手順 4 : View Model を いじる

Xamarin.Forms の view で、どのようにデータを表示するかのすべての機能を、`SpeakersViewModel.cs` が提供します。    
`SpeakersViewModel` は、 Speaker の List と、サーバからスピーカーのデータを取ってくるために呼ばれるメソッド、から構成されています。また、バックグラウンドタスクとしてデータを取って来ようとしていることを示す boolean フラグも持っています。   

### INotifyPropertyChanged をインプリメントしよう

`INotifyPropertyChanged` (= プロパティ値が変更されたことをクライアントに通知するインターフェース) は、 MVVM フレームワークにおいて、重要なデータバインディングで使用されます。 それ（INotifyPropertyChanged）は、オブジェクトの状態に変更があった場合に、ユーザーインターフェースが通知を受け取ることができる仕組みを提供します。

今こうなっています：

[Before]
```csharp
public class SpeakersViewModel
{

}
```

これを、こうしてください：

[After]【コピペ】
```csharp
public class SpeakersViewModel : INotifyPropertyChanged
{

}
```

そして `INotifyPropertyChanged` にオンマウスして Ctrl＋. (コントロールキーを押しながらピリオドキーを押す) → ［インターフェイスを実装します］を選択するか、`INotifyPropertyChanged` を右クリックすると表示される［クイックアクションとリファクタリング］をクリックし、［インターフェイスを実装します］をクリックします。

> macOS の場合は、オンマウスして Option + Enter →［インターフェイスを実装します］を選択するか、右クリックすると表示される［クイック修正］をクリックし、［インターフェイスを実装します］をクリックします。

![完成形](./image/quick_fix.png)

これで以下のコードが自動生成されます。

```csharp
public event PropertyChangedEventHandler PropertyChanged;
```

 `OnPropertyChanged` というヘルパーメソッド(helper method)を (`SpeakersViewModel` クラスの中に) 作りましょう。    
これは、プロパティが変更されるたびに呼ばれるメソッドです。

【コピペ】 C# 6
```csharp
void OnPropertyChanged([CallerMemberName] string name = null) =>
    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
```

(ちなみに、C# 5 (バージョン古い)で書くとこうなります)
```csharp
void OnPropertyChanged([CallerMemberName] string name = null)
{
    var changed = PropertyChanged;

    if (changed == null)
        return;

    changed.Invoke(this, new PropertyChangedEventArgs(name));
}
```

というわけで、
プロパティが更新されるたびに `OnPropertyChanged()` を呼ぶことができるようになりました。    
さぁ、私たちの最初の、バインディング可能なプロパティ (our first bindable property) を作ってみましょう！

### IsBusy プロパティ

この `SpeakersViewModel` クラスの中に、bool値を get/set するための、バッキングフィールド(backing field)と、bool型プロパティのためのアクセサを作りましょう。

まず、バッキングフィールドを作ります。

```csharp
bool isBusy;
```

そして、自動プロパティを作ります。

```csharp
public bool IsBusy
{
    get { return isBusy; }
    set
    {
        isBusy = value;
        OnPropertyChanged();
    }
}
```

`OnPropertyChanged();` を呼んでいますね。これを呼ぶことによって Xamarin.Forms は、IsBusy の値が set された時に、自動的に知ることができます。

【確認】現在、`SpeakersViewModel.cs` ファイルは次のようになっているはずです。

```csharp
using System.ComponentModel;
//...(略)
using System.Runtime.CompilerServices;

namespace DevDaysSpeakers.ViewModel
{
    public class SpeakersViewModel : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        bool isBusy;
        public bool IsBusy
        {
            get { return isBusy; }
            set
            {
                isBusy = value;
                OnPropertyChanged();
            }
        }

        void OnPropertyChanged([CallerMemberName] string name = null) =>
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
    }
}
```

### Speaker の ObservableCollection

[メモ]    
`ObservableCollection` (自分の中身が変わったことを検知する仕組みを持っているコレクション)

インターネットから取得してきた speaker のデータを格納するために `ObservableCollection` を使います。なぜ `ObservableCollection` を使うかというと、これは要素(コレクションの中身)を追加とか削除とかすると発火する `CollectionChanged` というイベントを最初から持っているからです。

コンストラクタを用意して、そのすぐ上に、自動プロパティを作ります。
>【スニペット】ちなみに「ctor」と打って tab + tab すると、コンストラクタを自動で生成してくれます)

```csharp
public ObservableCollection<Speaker> Speakers { get; set; } = new ObservableCollection<Speaker>();

public SpeakersViewModel()
{
}
```


### GetSpeakers メソッド

私たちは今から `GetSpeakers` という名前のメソッドを作ろうとしていますが、これは、インターネットから speaker のデータをすべて取ってくるときに呼ぶメソッドです。    
まずは単純な HTTP リクエストから実装します。でもあとから Azure からデータを取ってきたりこちら(クライアント)から変更して sync できるようにアップデートします！

まず、`GetSpeakers`という名前のメソッドを作ります。型は `async Task` です。

```csharp
async Task GetSpeakers()
{

}
```

これから、この `GetSpeakers` メソッドの中に色々とコードを書いていきます。

まず、既にデータを取得済みかをチェックします。

```csharp
async Task GetSpeakers()
{
    if(IsBusy)
        return;
}
```
次に、 try/catch/finally ブロックのための土台をいくつか作ります。

```csharp
async Task GetSpeakers()
{
    if (IsBusy)
        return;

    try
    {
        IsBusy = true;

    }
    catch (Exception e)
    {

    }
    finally
    {
       IsBusy = false;
    }
}
```

`IsBusy` は最初は true にセットしておき、そしてサーバーに接続しようとした時と、終わった時は false にセットします。

そして、`try`ブロックの中で、サーバーから json データを取ってくるために `HttpClient` を使います。

```csharp
using(var client = new HttpClient())
{
    // サーバーから json を取得します
    var json = await client.GetStringAsync("http://demo4404797.mockable.io/speakers");
}
```

引き続き **using** 句の中で、Json.NET を使用して json をデシリアライズし、Speaker のリストに格納します:

```csharp
var items = JsonConvert.DeserializeObject<List<Speaker>>(json);
```

引き続き **using** 句の中で、Speakers をクリアして、リスト内の item を ObservableCollection に読み込ませます:

```csharp
Speakers.Clear();
foreach (var item in items)
    Speakers.Add(item);
```

何か問題があれば **catch** 句でポップアップアラートを表示します:

```csharp
Debug.WriteLine("Error: " + e.Message);
await Application.Current.MainPage.DisplayAlert("Error!", e.Message, "OK");
```

`GetSpeakers` メソッドの完全なコードは次のようになるはずです:

```csharp
async Task GetSpeakers()
{
    if (IsBusy)
        return;

    try
    {
        IsBusy = true;

        using(var client = new HttpClient())
        {
            //サーバーから json を取得します
            var json = await client.GetStringAsync("http://demo4404797.mockable.io/speakers");

            //json をデシリアライズします
            var items = JsonConvert.DeserializeObject<List<Speaker>>(json);

            //リストを Speakers に読み込ませます
            Speakers.Clear();
            foreach (var item in items)
                Speakers.Add(item);
        }
    }
    catch (Exception e)
    {
        Debug.WriteLine("Error: " + e.Message);
        await Application.Current.MainPage.DisplayAlert("Error!", e.Message, "OK");
    }
    finally
    {
        IsBusy = false;
    }
}
```

データを取得するメインのメソッドはこれで完成です！

#### GetSpeakers Command

このメソッドを直接実行する代わりに、私たちは **Command** を公開します。Command は実行されるメソッドを知り、さらに、任意の Command が実行可能かどうかの分かるインターフェイスを持っています。

`ObservableCollection<Speaker> Speakers {get;set;}` を作成した場所で、**GetSpeakersCommand** Command を作成しましょう:

```csharp
public Command GetSpeakersCommand { get; }
```

**SpeakersViewModel()** のコンストラクターの中で GetSpeakersCommand を作成し、使用するメソッドを割り当てます。更に IsBusy プロパティに影響を与える enabled フラグも渡します:

```csharp
GetSpeakersCommand = new Command(
                async () => await GetSpeakers(), // 実行する処理
                () => !IsBusy); // このコマンドが実行可能の時にtrueを返す
```

この後修正するのは、私たちが作成した enabled 関数 (= Commandの第二引数) を再評価する IsBusy プロパティをいつセットするか？です。**IsBusy** の **set** でシンプルに **ChangeCanExecute** メソッドを次のように実行します:

```csharp
set
{
    isBusy = value;
    OnPropertyChanged();
    // CanExecute （このコマンドが実行可能かどうか）を更新
    GetSpeakersCommand.ChangeCanExecute();
}
```

【確認】現在、`SpeakersViewModel.cs` ファイルは次のようになっているはずです。

```csharp
using System.ComponentModel;
//...(略)
using System.Runtime.CompilerServices;

namespace DevDaysSpeakers.ViewModel
{
    public class SpeakersViewModel : INotifyPropertyChanged
    {
        public ObservableCollection<Speaker> Speakers { get; set; } = new ObservableCollection<Speaker>();
        public Command GetSpeakersCommand { get; }

        public SpeakersViewModel()
        {
            GetSpeakersCommand = new Command(
                async () => await GetSpeakers(),
                () => !IsBusy);
        }

        public event PropertyChangedEventHandler PropertyChanged;

        bool isBusy;
        public bool IsBusy
        {
            get { return isBusy; }
            set
            {
                isBusy = value;
                OnPropertyChanged();
            }
        }

        // インターネットから speaker のデータをすべて取ってくる
        async Task GetSpeakers()
        {
            if (IsBusy)
                return;

            try
            {
                IsBusy = true;

                using(var client = new HttpClient())
                {
                    //サーバーから json を取得します
                    var json = await client.GetStringAsync("http://demo4404797.mockable.io/speakers");

                    //json をデシリアライズします
                    var items = JsonConvert.DeserializeObject<List<Speaker>>(json);

                    //リストを Speakers に読み込ませます
                    Speakers.Clear();
                    foreach (var item in items)
                        Speakers.Add(item);
                }
            }
            catch (Exception e)
            {
                //ポップアップアラートを表示します。
                Debug.WriteLine("Error: " + e.Message);
                await Application.Current.MainPage.DisplayAlert("Error!", e.Message, "OK");
            }
            finally
            {
                IsBusy = false;
            }
        }

        void OnPropertyChanged([CallerMemberName] string name = null) =>
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
    }
}
```

## ユーザーインターフェース!!!

さて、最初の Xamarin.Forms ユーザーインタフェースとして、**View/SpeakersPage.xaml** を作っていきましょう。

今までは C# で書いてきましたが、ここから(ユーザインタフェース)は **XAML** (ざむる) という、マークアップ言語で書いていきます。(Extensible Application Markup Language)

### SpeakersPage.xaml

アプリの最初のページとして、縦にスタックされる(縦に連なって表示される)コントロール群を追加します。はじめに、ContentPage の中に、次のように StackLayout を追加します：

```xml
 <StackLayout Spacing="0">

 </StackLayout>
```

今後、この StackLayout に対して、他のコントロールを追加していきます。

次に、既に作成した **GetSpeakersCommand** にバインディングされるボタンを追加します：

```xml
<Button Text="Sync Speakers" Command="{Binding GetSpeakersCommand}"/>
```

このボタンの下に、サーバーからデータを受信しているときに使うローディングバーを表示します。ここでは、ActivityIndicator を使って、既に作ってある IsBusy プロパティにバインディングします：

```xml
<ActivityIndicator IsRunning="{Binding IsBusy}" IsVisible="{Binding IsBusy}"/>
```

【確認】現在、`SpeakersPage.xaml`は、次のようになっているはずです。

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="DevDaysSpeakers.View.SpeakersPage"
             Title="Speakers">
    <StackLayout Spacing="0">
        <Button Text="Sync Speakers" Command="{Binding GetSpeakersCommand}"/>
        <ActivityIndicator IsRunning="{Binding IsBusy}" IsVisible="{Binding IsBusy}"/>
    </StackLayout>
</ContentPage>
```

Speakers コレクションに ListView をバインディングして、全ての要素を表示します。
また、*x:Name=""* という特別なプロパティを使って、コントロールに名前を付けることができます。ここでは、ListView に ListViewSpeakers という名前を付けています。先程の ActivityIndicator の下に書きましょう：

```xml
<ListView x:Name="ListViewSpeakers"
              ItemsSource="{Binding Speakers}">
        <!--ここに ItemTemplate を追加-->
</ListView>
```

要素の表示には、個々の要素がどのように表示されるべきかを記述する必要があります。そのためには、 ItemTemplate を使いますが、この ItemTemplate は、何らかの View で表現された DataTemplate を持つ必要があります。
Xamarin.Forms には、いくつかの既定の Cell が定義されていますが、ここでは、一つの画像と、2つのテキストを表示できる、**ImageCell** を使います。

上記のコード内の &lt;!--ここに ItemTemplate を追加--&gt; を、次のコードで置き換えます：

```xml
<ListView.ItemTemplate>
    <DataTemplate>
        <ImageCell Text="{Binding Name}"
                    Detail="{Binding Title}"
                    ImageSource="{Binding Avatar}"/>
    </DataTemplate>
</ListView.ItemTemplate>
```

Xamarin.Forms の ImageCell は、サーバー上の画像を自動的にダウンロードして、キャッシュした後に、表示します。

### View に ViewModel を接続する

View の要素に ViewModel の プロパティをバインディングしてきましたが、この View にどの ViewModel をバインディングするのか教える必要があります。つまり、 `SpeakersViewModel` を作成して `BindingContext` に設定します。 `SpeakersPage.xaml.cs` を開いてみてください。必要なコードは書いておきました。

```csharp
SpeakersViewModel vm;

public SpeakersPage()
{
    InitializeComponent();

    // Create the view model and set as binding context
    vm = new SpeakersViewModel();
    BindingContext = vm;
}
```

### App.cs を確認する

App.cs を開いてみると、そこには、App() のコンストラクタがあり、そこがアプリケーションのエントリーポイントになっています。この中では、新しい SpeackersPage を作成し、それをナビゲーション ページでラップすることで、タイトルバーを作っています。

### アプリを実行する!

遂に、iOS、Android、あるいは、UWP (Windows/VS2015 のみ) をスタートアップ プロジェクトとして設定し、デバッグを開始することができるようになりました！

![Startup project](./image/AppRun001.png)

![実行する](./image/jikkou.png)

#### iOS
PCを使っている場合、アプリの実行・デバッグを行うためには、XamarinがインストールされているmacOSのデバイスに接続する必要があります。

macOSに正しく接続されている場合、接続状態は緑になっています。ターゲットとして、 **iPhoneSimulator** を選択してから、デバッグを行うシミュレータの種類を選択します。

![iOS Setup](./image/AppRun002.png)

> [メモ]    
[iOS Simulator (for Windows) - Xamarin](https://developer.xamarin.com/guides/cross-platform/windows/ios-simulator/) を使用すると、Windows 側に iOS Simulator の画面が転送されます。


#### Android

DevDaysSpeakers.Droid をスタートアップ プロジェクトとして設定し、実行するエミュレーターを選択します。

#### Windows 10

DevDaysSpeakers.UWP をスタートアップ プロジェクトとして設定し、［ソリューションプラットフォーム］から **x86** を、デバッグ対象から **ローカル コンピューター** を選択します。

> メモ：元のドキュメントでは SQLite 拡張機能をインストールしていますが、インストールしなくても動作するようです。<br />
<br />
最初に、UWP アプリ用に SQLite 拡張がインストールされていることを確認します：        
**ツール->拡張機能と更新プログラム** に移動します。    
オンラインの検索で、*SQLite* を検索し、SQlite for Univeral Windows Platform がインストールされていることを確認します。(執筆時のバージョンは、 3.13.0)    
![Sqlite](http://content.screencast.com/users/JamesMontemagno/folders/Jing/media/ace42b1e-edd8-4e65-92e7-f638b83ad533/2016-07-11_1605.png)

/* ハンズオン運営メモ：ここで休憩 and 必要なら講師交代 */

## Details (詳細画面)

さて、次に、このページから、ナビゲーションを経て、何らかの Details (詳細画面)を表示してみましょう。
**SpeakersPage.xaml** に対するコード ビハインドである、**SpeakersPage.xaml.cs** を開きます。

### ItemSelected イベント

コード ビハインドの中には、SpeakersViewModel に対する設定があります。項目が選択されたときに通知されるように、**BindingContext = vm;** の下に、**ListViewSpeakers** へのイベントを追加します。

```csharp
ListViewSpeakers.ItemSelected += ListViewSpeakers_ItemSelected;
```


そして、DetailsPage へナビゲートされるように、ListViewSpeakers_ItemSelected を作成します：

```csharp
private async void ListViewSpeakers_ItemSelected(object sender, SelectedItemChangedEventArgs e)
{
    var speaker = e.SelectedItem as Speaker;
    if (speaker == null)
        return;

    await Navigation.PushAsync(new DetailsPage(speaker));

    ListViewSpeakers.SelectedItem = null;
}
```

上のコードでは、最初に選択されている項目が null でないことを確認してから、**Navigation** API を使って、新しく作ったページをプッシュし、最後に、項目の選択を解除します。

### DetailsPage.xaml

さて、Details ページを作っていきます。
SpeakersPage と同じように StackLayout を使いますが、ここでは、テキストが長すぎる場合にスクロールできるように、StackLayout を ScrollView で囲みます。

```xml
  <ScrollView Padding="10">
    <StackLayout Spacing="10">
     <!-- 詳細画面のコントロール群をここに書く -->
    </StackLayout>    
  </ScrollView>
```

そして、Speaker オブジェクトのプロパティに対応するコントロールと、バインディングを追加します：

```xml
<Image Source="{Binding Avatar}" HeightRequest="200" WidthRequest="200"/>

<Label Text="{Binding Name}" FontSize="24"/>
<Label Text="{Binding Title}" TextColor="Purple"/>
<Label Text="{Binding Description}"/>
```

さらに、ちょっとだけ面白くするために、2つのボタンを追加します。これらのボタンには、この後、コード ビハインドでクリック時のイベントを追加します：

```xml
<Button Text="読み上げる" x:Name="ButtonSpeak"/>
<Button Text="ウェブサイトに移動" x:Name="ButtonWebsite"/>
```

【確認】結果的に、`DetailsPage.xaml`は このようになっているはずです。

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="DevDaysSpeakers.View.DetailsPage"
             Title="Details">
    <ScrollView Padding="10">
        <StackLayout Spacing="10">
            <!-- 詳細画面のコントロール群をここに書く -->
            <Image Source="{Binding Avatar}" HeightRequest="200" WidthRequest="200"/>

            <Label Text="{Binding Name}" FontSize="24"/>
            <Label Text="{Binding Title}" TextColor="Purple"/>
            <Label Text="{Binding Description}"/>

            <Button Text="読み上げる" x:Name="ButtonSpeak"/>
            <Button Text="ウェブサイトに移動" x:Name="ButtonWebsite"/>
        </StackLayout>
    </ScrollView>
</ContentPage>
```

### 読み上げる

**DetailsPage.xaml.cs** を開いて、2つのクリック ハンドラを足しますが、ここでは、ButtonSpeak から始めます。
このハンドラでは、 [Xamarin.Essentials](https://docs.microsoft.com/en-us/xamarin/essentials/) の [Text-To-Speech](https://docs.microsoft.com/en-us/xamarin/essentials/text-to-speech) を使って、スピーカーの詳細を読み上げるようにします。

コンストラクタの BindingContext の下で、クリック ハンドラを追加します：

```csharp
ButtonSpeak.Clicked += ButtonSpeak_Clicked;
```

クリック ハンドラでは、Text To Speech のクロス プラットフォームな API を呼び出します：

```csharp
async void ButtonSpeak_Clicked(object sender, EventArgs e)
{
    await TextToSpeech.SpeakAsync(this.speaker.Description);
}
```

### ウェブサイトに移動
Xamarin.Forms には、URL を既定のブラウザで開くためのクロス プラットフォームな機能が搭載されています。

今度は、ButtonWebsite のクリック ハンドラを追加します：

```csharp
ButtonWebsite.Clicked += ButtonWebsite_Clicked;
```

そして、Device キーワードを用いて、OpenUri メソッドを呼び出します：

```csharp
void ButtonWebsite_Clicked(object sender, EventArgs e)
{
    if (speaker.Website.StartsWith("http"))
        Device.OpenUri(new Uri(speaker.Website));
}
```

### コンパイル & 実行

ここまでできたら、残りの微調整をして、前と同じようにコンパイル、実行できるようにします。実行画面は次のようになります。

**Android**

<img src="image/Run_Android01.png" width="300" /> <img src="image/Run_Android02.png" width="300" />

**iOS**

<img src="image/Run_iOS01.png" width="300" /> <img src="image/Run_iOS02.png" width="300" />

**UWP**

<img src="image/Run_UWP01.png" width="300" /> <img src="image/Run_UWP02.png" width="300" />

## Azure Mobile Apps に接続します

もちろん RESTful なエンドポイントからデータを取得できることは大事ですがフル機能のバックエンドについてはどうでしょうか？そこで Azure Mobile Apps ですよ！それでは私たちのアプリケーションを Azure Mobile Apps バックエンドを使うようにアップグレードしていきましょう。

[http://portal.azure.com](http://portal.azure.com) にアクセスし、アカウントを取得します。

ポータルにアクセスしたら、**+ 新規** ボタンを選択し、**mobile apps** を検索します。下の図のように検索結果が表示されますので、**Mobile Apps Quickstart** を選択します。

![Quickstart](image/ConnectAzure_Quickstart.png)

Quickstart のブレードが開くので、**作成** をクリックします。

<img src="image/ConnectAzure_CreateQuickstart.png" width="500" />

4つの設定項目がある設定ブレードが開きます:

**アプリ名**

これはアプリのバックエンドを設定するのに必要な一意のアプリ名です。グローバルで一意の名前が必要です。例えば *yourlastnamespeakers* などを試してみてください。

**サブスクリプション**

サブスクリプションを選択するか、従量課金のアカウントを作成します。(このサービスでは課金は発生しません)

**リソースグループ**

*新規作成* を選択し、**DevDaysSpeakers** と入力します。

リソースグループは後で関連のあるサービスを一度に簡単に削除するためのグループです。

**App Service プラン/場所**

このフィールドをクリックして **新規作成** を選択し、一意の名前を付けます。場所 (通常は近い場所を選択します) を選択し、F1 Free の価格レベルを選択します:

> メモ: **価格レベルを選択** ブレードの右上にある **すべて表示** をクリックすると F1 Free の価格レベルが表示され選択できるようになります。

![service plan](image/ConnectAzure_ServicePlan.png)

最後に **ダッシュボードにピン留めする** をチェックし、［作成］をクリックします:

![Create](image/ConnectAzure_Create.png)

Mobile Apps のセットアップが完了するまでに 3～5 分ほど掛かります。コードに戻りましょう！

Azure バックエンドを私たちのモバイルアプリに追加するために、更にコードを修正していきます。

### AzureService.cs のアップデート

**Service/AzureService.cs** を開き、必要な編集していきましょう。

**Initialize()** メソッド内の appUrl の **OUR-APP-NAME-HERE** を作成した Azure Mobile Apps の名前で書き換えます。手順の通りにやっていれば "https://xxxxspeakers.azurewebsites.net" になるはずです。

次に **GetSpeakers()** メソッドを table を初期化して同期し、Nameを基準に昇順で並び替える以下の行で置き換えましょう:

```csharp
await Initialize();
await SyncSpeakers();
return await table.OrderBy(s => s.Name).ToEnumerableAsync();
```

次に **SyncSpeakers()** の try 句にローカルの table をプッシュして同期する以下の行を追加しましょう:

```csharp
await Client.SyncContext.PushAsync();
await table.PullAsync("allSpeakers", table.CreateQuery());
```

AzureService.cs の編集はこれで終了です。

### SpeakersViewModel.cs のアップデート

次に **ViewModel/SpeakersViewModel.cs** を編集していきましょう。

**GetSpeakers()** メソッドの try/catch 句で JSON を取得していた **using** 句をすべて削除します。

そのまま try 句で先ほど修正した **AzureService** を Dependency Service としてインスタンス化し、Azure Mobile Apps の table から Speakers のデータを取得する以下の行を追加しましょう:

```csharp
var service = DependencyService.Get<AzureService>();
var items = await service.GetSpeakers();

Speakers.Clear();
foreach (var item in items)
    Speakers.Add(item);
```

try 句は次のようになります。

```csharp
try
{
    IsBusy = true;

    var service = DependencyService.Get<AzureService>();
    var items = await service.GetSpeakers();

    Speakers.Clear();
    foreach (var item in items)
        Speakers.Add(item);
}
```


これで私たちのアプリに必要な実装がすべて終了しました！

Azure portal に戻りデータベースを見てみましょう。

Quickstart が終了したら、以下の画面が見えるはずです。または、ダッシュボードのピンをタップしても行けます:

<img src="image/ConnectAzure_Dashboard.png" width="500" />

**Features** 内の **Easy Tables** を選択します。

作成済みの todoitem が見えるはずですが、ここでは新しいテーブルを作成し、デフォルトのデータセットを **Add from CSV** からアップロードすることで作成します。

このレポジトリをダウンロードし、**Speaker.csv** がフォルダにあることを確認してください。

ファイルを選択すると、新しいテーブル名が追加され、フィールドを探して追加してくれます。その後、Start Upload をクリックします。

![upload data](image/ConnectAzure_AddCsv.png)

一度アプリケーションを削除して、再度実行してみましょう(アプリ内に前回までの table データが残っており、最初に PUSH するロジックのため二重に table データが登録されるのを防ぐため)。Azure のデータが取得できているはずです！


## 宿題

二つの宿題でDev Daysをさらに進めましょう。

### 宿題1: Cognitive Services

>【メモ】2018/11/16 現在は　Cognitive Service の [Face](https://azure.microsoft.com/ja-jp/services/cognitive-services/face/) を使用します。

<s>

[Cognitive Serivce Emotion API](https://www.microsoft.com/cognitive-services/en-us/emotion-api)を使い、詳細ページに話し手の表情から幸福度を解析するボタンを追加しましょう。

![Microsoft.ProjectOxford.Emotion](image/withEmotion.png)


http://microsoft.com/cognitive からアカウントとAPIキーを取得し、以下の手順を踏んでください。

#### Cognitive Serivce Emotion API のアカウント作成

上記リンクから Web にアクセスします。Microsoft アカウントでログインし、**Get started for free** をクリックします。

![Cognitive Serivce Emotion API](image/Cognitive_Emotion01.png)


(もしうまく行かなかったらの時の話)
![Cognitive Serivce Emotion API](image/GotErrorWhenRegisteringEmotionAPI.png)


画面が遷移します。**Emotion - Preview** にチェックが入っていることを確認し、画面下の Term、Privacy Policy のチェックをオンにして、**Subscribe** をクリックします。(Contact me with promotional offers and updates about Microsoft Cognitive Services. はチェックしなくても構いません)

![Cognitive Serivce Emotion API](image/Cognitive_Emotion02.png)

**My free subscriptions** に **Emotion** が登録されているので、**Show** をクリックして、Key1 のキーを控えておきます。

![Cognitive Serivce Emotion API](image/Cognitive_Emotion03.png)

#### Visual Studio での作業

1.) **Microsoft.ProjectOxford.Emotion** を全プロジェクト(一番上の共通プロジェクト以外)に追加します。

↓ Visual Studio for Mac での NuGet Package の追加方法
![Microsoft.ProjectOxford.Emotion](image/AddingNuGetPackage.png)

![Microsoft.ProjectOxford.Emotion](image/Cognitive01.png)

2.) `EmotionService`クラスを追加します (GetHappinessAsync の中の API キーは直してください)

必要な using は以下の通りです。

```csharp
using Microsoft.ProjectOxford.Emotion;
using Microsoft.ProjectOxford.Emotion.Contract;
using System;
using System.Linq;
using System.Net.Http;
using System.Threading.Tasks;
```

以下のクラスを作成します。

```csharp
// MSの エモーションAPIサービスを使い、スピーカーの顔写真の幸せ度(どの程度笑顔か)を判断するクラス
public class EmotionService
{
    private static async Task<Emotion[]> GetHappinessAsync(string url)
    {
        var emotionClient = new EmotionServiceClient("ここにAPIキー文字列を入れてね");

        var emotionResults = await emotionClient.RecognizeAsync(url);

        if (emotionResults == null || emotionResults.Count() == 0)
        {
            // 顔写真で人間の顔が認識できなかった場合(猿とか)は例外を吐いて落ちる
            throw new Exception("顔が認識できないよ");
        }

        return emotionResults;
    }

    //複数の被検対象が存在する場合の平均幸福度算出
    public static async Task<float> GetAverageHappinessScoreAsync(string url)
    {
        Emotion[] emotionResults = await GetHappinessAsync(url);

        float score = 0;
        foreach (var emotionResult in emotionResults)
        {
            score = score + emotionResult.Scores.Happiness;
        }

        return score / emotionResults.Count();
    }

    // 幸福度スコア(大きいほど笑顔)を受け取り、その評価文字列を返す
    public static string GetHappinessMessage(float score)
    {
        score = score * 100;
        double result = Math.Round(score, 2);

        if (score >= 50)
            return result + " % ヽ（ヽ *ﾟ▽ﾟ*）ノわーい！しあわせ！";
        else
            return result + "% （；＿；）しあわせじゃない";
    }
}
```

3.) 詳細ページにボタンを追加し、 **x:Name="ButtonAnalyze"** と指定します。

4.) クリックのハンドラを追加し、`async`キーワードを指定します。

5.) 以下のコードを追加します。

```csharp
var level = await EmotionService.GetAverageHappinessScoreAsync(this.speaker.Avatar);
```

6.) ポップアップアラートを表示します。

```csharp
await DisplayAlert("Happiness Level", EmotionService.GetHappinessMessage(level), "OK");
```

完成！
![Microsoft.ProjectOxford.Emotion](image/withEmotion.png)


</s>

### 宿題2: 話し手の詳細を編集する

ここでは話し手の肩書きを編集可能にします。

`DetailsPage.xaml`を開き、肩書きを表示している`Label`

```xml
<Label Text="{Binding Title}" TextColor="Purple"/>
```

を以下のような`OneWay`データバインディングを持つ`Entry`に変更し、`Name`を指定してください。

`OneWay`にすると、テキストを入力しても実際のデータは変更されません。

```xml
<Entry Text="{Binding Title, Mode=OneWay}"
             TextColor="Purple"
             x:Name="EntryTitle"/>
```

保存ボタンを「ウェブサイトに移動」ボタンの下に追加しましょう。

```xml
<Button Text="保存" x:Name="ButtonSave"/>
```

#### SpeakersViewModel を更新する

`AzureService.cs`を開き、話し手を同期し、リストを更新する`UpdateSpeaker(Speaker speaker)`メソッドを追加します。

```csharp
public async Task UpdateSpeaker(Speaker speaker)
{
    await Initialize();
    await table.UpdateAsync(speaker);
    await SyncSpeakers();
}
```

`SpeakersViewModel.cs` を開き、同じようなメソッドを追加します:

```csharp
public async Task UpdateSpeaker(Speaker speaker)
{
    var service = DependencyService.Get<AzureService>();
    await service.UpdateSpeaker(speaker);
    await GetSpeakers();         
}
```


#### DetailsPage.xaml.csを更新する

SpeakersViewModelを受け取るようにDetailsPageのコンストラクタを変更しましょう。

変更前:
```csharp
Speaker speaker;
public DetailsPage(Speaker speaker)
{
    InitializeComponent();
    this.speaker = speaker;
    ...
}
```
変更後:
```csharp
Speaker speaker;
SpeakersViewModel vm;
public DetailsPage(Speaker item, SpeakersViewModel viewModel)
{
    InitializeComponent();
    this.speaker = item;
    this.vm = viewModel;
    ...
}
```

`ButtonSave`のためのハンドラを追加します。

```csharp
ButtonSave.Clicked += ButtonSave_Clicked;
```

ボタンがクリックされたとき、話し手を更新・保存し前のページに戻ります。

```csharp
private async void ButtonSave_Clicked(object sender, EventArgs e)
{
    speaker.Title = EntryTitle.Text;
    await vm.UpdateSpeaker(speaker);
    await Navigation.PopAsync();
}
```

最後に、**SpeakersPage.xaml.cs** の `ListViewSpeakers_ItemSelected`で`SpeakersPage.xaml.cs`へ飛ぶときにViewModelを渡すようにする必要があります。

```csharp
//ビューモデルを渡す。
await Navigation.PushAsync(new DetailsPage(speaker, vm));
```

**保存** ボタンをタップすると、Mobile Apps の Easy Table の **Title** 列を修正します。

![Modified](image/save01.png)

できあがり！
