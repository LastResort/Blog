# 關於Javascript檔在Web form的MasterPage引用上的狀況
紀錄在MasterPage使用相對路徑引用js，於不同層級的page套用時其路徑的變化。

## 於MasterPage使用相對路徑

以下是Site1.master的原始碼，請注意`head`並未使用`runat`這個屬性。

```aspx
<%@ Master Language="C#" AutoEventWireup="true" CodeBehind="Site1.master.cs" Inherits="WebApplication1.Site1" %>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
     <%--使用了相對路徑，導致會回上一層--%>
    <script src="../Scripts/jquery-1.10.2.min.js"></script>
    
    <%--從根目路開始找起--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveUrl--%>
    <%--如果參數是絕對路徑，回傳值將原封不動傳回；若是相對路徑，則會變更為適用於目前的要求路徑，以便在瀏覽器可以正確無誤的解析URL的相對路徑。--%>
    <script src="<%= ResolveUrl("~/Scripts/jquery-1.10.2.min.js") %>"></script>

    <%--System.Web.UI.Control.ResolveClientUrl--%>
    <%--在MSDN中註解寫到：這個方法所傳回的 URL 會相對於包含控制項具現化的原始程式檔的資料夾。 繼承這個屬性，例如控制項 UserControl 和 MasterPage, ，將會傳回相對於控制項的完整的 URL
    簡單來說就是依據原始程式檔的位置回傳其相對位置--%>
    <script src="<%= ResolveClientUrl("~/Scripts/jquery-1.10.2.min.js") %>"></script>
    </asp:ContentPlaceHolder>
</head>
...
</html>
```

接下來分別使用四個層級的路徑來引用：
1. `localhost/root.aspx`
```html
    <%--使用了相對路徑，導致會回上一層，但因為是根目錄，所以不會有404--%>
    <script src="../Scripts/jquery-1.10.2.min.js"></script>
    
    <%--從根目路開始找起--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveUrl--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveClientUrl--%>
    <script src="Scripts/jquery-1.10.2.min.js"></script>
```

2. `localhost/first/first.aspx`
```html
    <%--使用了相對路徑，導致會回上一層--%>
    <%-- 200: http://localhost:65042/Scripts/jquery-1.10.2.min.js --%>
    <script src="../Scripts/jquery-1.10.2.min.js"></script>
    
    <%--從根目路開始找起--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveUrl--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveClientUrl--%>
    <script src="../Scripts/jquery-1.10.2.min.js"></script>
```

3. `localhost/first/second/second.aspx`
```html
    <%--使用了相對路徑，導致會回上一層--%>
    <%-- 404: http://localhost:65042/first/Scripts/jquery-1.10.2.min.js --%>
    <script src="../Scripts/jquery-1.10.2.min.js"></script>
    
    <%--從根目路開始找起--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveUrl--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveClientUrl--%>
    <script src="../../Scripts/jquery-1.10.2.min.js"></script>
```

4. `localhost/first/second/third/third.aspx`
```html
    <%--使用了相對路徑，導致會回上一層--%>
    <%-- 404: http://localhost:65042/first/second/Scripts/jquery-1.10.2.min.js --%>
    <script src="../Scripts/jquery-1.10.2.min.js"></script>
    
    <%--從根目路開始找起--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveUrl--%>
    <script src="/Scripts/jquery-1.10.2.min.js"></script>
    
    <%--System.Web.UI.Control.ResolveClientUrl--%>
    <script src="../../../Scripts/jquery-1.10.2.min.js"></script>
```

## 結論
在MasterPage底下引用js的話，建議是用`/Script/path`或者是用`ResolveUrl`這種由根目路往下查找的方式來引用會來的簡單明瞭。當然想用`ResolveClientUrl`當然也是可以，最不建議的當然就是用`../Script/`這種相對路徑的方式。

另外在javascript底下，當然就是用絕對路徑或是由根目錄查找的相對路徑方式來組URL。

