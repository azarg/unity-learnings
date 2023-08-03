```cs
public void SaveColor() {
    SaveData data = new SaveData();
    // initialize fields on "data"
    // data.someField = someValue;

    string json = JsonUtility.ToJson(data);
    File.WriteAllText(Application.persistentDataPath + "/savefile.json", json);
}

public void LoadColor() {
    string path = Application.persistentDataPath + "/savefile.json";
    if (File.Exists(path)) {
        string json = File.ReadAllText(path);
        SaveData data = JsonUtility.FromJson<SaveData>(json);

        // assign loaded values to variables
        // SomeVariable = data.someField;
    }
}
```
