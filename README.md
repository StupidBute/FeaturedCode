# 各專案的精選程式碼彙整

## 圖形化對話編輯器DialogueTree
### Dialogue類別
負責編輯器視窗的實際繪製，以及調用各個面版類別的點擊與資料設定/取得函式
    public class DialogueTree : EditorWindow {
      public enum WindowState{normal, drag, move, popup, link, scroll};
      public WindowState nowState = WindowState.normal;

      public LeftPanel leftPanel;
      public RightPanel rightPanel;
      ColorWindow colorWindow;

      public List<Character> lst_chars = new List<Character> ();
      public List<Node> lst_node = new List<Node> ();
      public Node SelectNode = null;
      public int plotNodeCount = 0;
      Texture2D tex_bg, tex_left, tex_add;
      GUIStyle style_button;
      Vector2 coordinate;

      [MenuItem("Window/Dialogue Tree")]
      static void Init(){
        DialogueTree window = (DialogueTree)GetWindow (typeof(DialogueTree));
        window.minSize = new Vector2 (400, 250);
        window.titleContent = new GUIContent ("Dialogue Tree");
        window.Show ();

      }

      void OnEnable(){
        tex_bg = Resources.Load<Texture2D> ("GUISkin/Grid");
        lst_chars.Add (new Character (this, "N/A", 7));
        coordinate = Vector2.zero;
        colorWindow = new ColorWindow ();
        leftPanel = new LeftPanel (this);
        rightPanel = new RightPanel (this);
        GUISkin mySkin = Resources.Load<GUISkin> ("GUISkin/NodeSkin");
        style_button = mySkin.GetStyle ("button");

        CreateNode (Vector2.zero, 0);
        //Selection.selectionChanged = LoadStoryAsset;
      }

      void OnGUI(){
        DrawBackground ();

        ProcessEvent (Event.current);

        DrawNodes ();

        DrawPanels ();

        Repaint ();
      }

## 遊戲專案《沉沒意志》
### 

## 遊戲專案《圖塔圖塔》
### 
