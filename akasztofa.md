# Grid: TicTacToe

## Feladat:

Solution neve: **akasztofa**

Projekt leírása: Hozz létre egy MainGrid-et, amiben hozz létre négy másik grid-et. bal felül a feladvány az "Akasztófa" címmel, bal alul a betűgombok, jobb felül a hibák száma, jobb alul az akasztófa frame-jeinek a helye 11 hibalehetőséggel.

Működés: A gombok megnyomásával, betűket tippelhetünk be, ha megjelenik bal felül a feladványban(a keresett szóban) akkor helyesen válasszoltunk, ha nem akkor jobb fent a hibás probálkozások 1-el megnövekszik és az akasztófa elkezd megépülni.  

1. Window méretét beállítjuk 450x800, Ha nem az a alapértelmezett.
2. Grid: Egy MainGrid-ben 4 másik Grid helyezkedik el.
3. Button-ok: Button-ként írjuk ki a kitalált betűket.
4. PuzzleGrid: Ez a rész felelős a játék címéért és a kitalálandó szó megjelenítéséért.
5. LetterGrid: Ez a terület tartalmazza az ábécé gombjait, amelyeket a játékos kattinthat.
6. InfoGrid: Ez a rész információkat jelenít meg a játékos számára
7. HangmanGrid: Itt jelenik meg a felakasztott figura (akasztófa) állapota.
```c#
<Window x:Class="akasztofa.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:akasztofa"
        mc:Ignorable="d"
        Title="Akasztófa" Height="450" Width="800">

    <!-- Fő Grid -->
    <Grid Name="MainGrid">
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="*" />
        </Grid.ColumnDefinitions>

        
        <Grid Name="PuzzleGrid" Grid.Row="0" Grid.Column="0" Background="LightGray">
            <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
                <TextBlock Text="Akasztófa" FontSize="20" FontWeight="Bold" HorizontalAlignment="Center"/>
                <WrapPanel Name="WordDisplay" HorizontalAlignment="Center"/>
            </StackPanel>
        </Grid>
      
        <Grid Name="LetterGrid" Grid.Row="1" Grid.Column="0" Background="LightGray">
            <UniformGrid Name="LetterButtons" Columns="9" Rows="4" Margin="10">
                <!-- buttons -->
               
            </UniformGrid>
        </Grid>
       
        <Grid Name="InfoGrid" Grid.Row="0" Grid.Column="1" Background="LightGray">
            <TextBlock Name="InfoText" Text="Hibás próbálkozások: 0" FontSize="16" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </Grid>

        <Grid Name="HangmanGrid" Grid.Row="1" Grid.Column="1" Background="Transparent">
            <Image Name="HangmanImage" Width="157" Height="81" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </Grid>
    </Grid>
</Window>


```
## MainWindow.xaml.cs 

8. Szavak betöltése
A program betölt egy fájlból (`szavak.txt`) véletlenszerű szót.
A fájl minden sorában egy szó található, amelynek hossza legfeljebb 20 karakter lehet. A szóban nem szerepelhet szóköz vagy kötőjel.
```c#
private void LoadWords()
{
    try
    {
        string[] words = File.ReadAllLines("szavak.txt"); // Szavak beolvasása fájlból

        // A szavak szűrése: max. 20 karakter, nem tartalmaz szóközt vagy kötőjelet
        words = words.Where(w => w.Length <= 20 && !w.Contains(" ") && !w.Contains("-")).ToArray();

        if (words.Length == 0)
        {
            MessageBox.Show("A szavak.txt fájlban nincs megfelelő szó.");
            Application.Current.Shutdown(); // Ha nincs érvényes szó, kilép a program
        }

        word = words[rnd.Next(words.Length)].ToUpper(); // Véletlenszerű szó választása
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Hiba történt a szavak betöltésekor: {ex.Message}");
        Application.Current.Shutdown(); // Hiba esetén kilép a program
    }
}
```
9. A játék inicializálása (StartNewGame metódus)
Ez a metódus felelős a játék újraindításáért, amely beállítja az új szót, a próbálkozások számát, és elindítja a szükséges UI elemeket:

```c#
private void StartNewGame()
{
    LoadWords(); // Új szavak betöltése
    guessedLetters = new List<char>(new string('_', word.Length).ToCharArray()); // Kitalált betűk kezdeti állapota
    attemptsLeft = 11; // A próbálkozások száma 11
    guessedChars.Clear(); // A már kitalált karakterek listájának törlése
    CreateWordButtons(); // Szó gombok létrehozása
    UpdateInfoText(); // Információ frissítése
    UpdateHangmanImage(); // Hangman kép frissítése
    CreateLetterButtons(); // Betű gombok létrehozása
}

```
10. A szó gombok létrehozása (CreateWordButtons metódus)
Ez a metódus hozza létre a szóhoz tartozó gombokat a képernyőn:
```c#
private void CreateWordButtons()
{
    WordDisplay.Children.Clear(); // Törli a szó gombjait
    wordButtons.Clear(); // A gombok listájának törlése

    for (int i = 0; i < word.Length; i++)
    {
        Button btn = new Button
        {
            Content = "_", // Kezdeti állapot: _
            Tag = word[i].ToString(), // A gombhoz rendelt betű
            Name = "LetterButton" + i, // Gomb név
            FontSize = 16, // Betűméret
            IsEnabled = false // A gomb le van tiltva
        };
        wordButtons.Add(btn); // A gomb hozzáadása a listához
        
        WordDisplay.Children.Add(btn); // A gomb hozzáadása a felhasználói felülethez
    }
}

```
10. Betű próbálgatása (LetterButton_Click metódus)
Ez a metódus kezeli a felhasználó által kiválasztott betűt, és frissíti a játék állapotát:
```c#
private void LetterButton_Click(object sender, RoutedEventArgs e)
{
    Button button = (Button)sender; // Az éppen kattintott gomb
    char guessedChar = button.Content.ToString()[0]; // A kitalált betű
    button.IsEnabled = false; // A gomb letiltása

    if (word.Contains(guessedChar))
    {
        // Ha a betű benne van a szóban, frissítjük a megfelelő gombokat
        for (int i = 0; i < word.Length; i++)
        {
            if (word[i] == guessedChar)
            {
                wordButtons[i].Content = guessedChar.ToString(); // A betű megjelenítése a megfelelő gombon
            }
        }
    }
    else
    {
        attemptsLeft--; // Hibás próbálkozás, csökkentjük a próbálkozások számát
    }

    UpdateInfoText(); // Információ frissítése
    UpdateHangmanImage(); // Hangman kép frissítése

    // Játék vége: ha elfogytak a próbálkozások vagy kitalálták a szót
    if (attemptsLeft == 0)
    {
        MessageBox.Show($"Vesztettél! A szó a következő volt: {word}");
        StartNewGame(); // Új játék indítása
    }
    else if (!wordButtons.Any(btn => btn.Content.ToString() == "_"))
    {
        MessageBox.Show("Gratulálok, nyertél!");
        StartNewGame(); // Új játék indítása
    }
}

```
11. Hangman kép frissítése (UpdateHangmanImage metódus)
Ez a metódus frissíti a hangman képet a próbálkozások számának megfelelően:

```c#
private void UpdateHangmanImage()
{
    int errorCounter = 11 - attemptsLeft; // A próbálkozások számától függően
    HangmanImage.Source = new BitmapImage(new Uri($"Images/akasztofa{errorCounter}.png", UriKind.Relative)); // A megfelelő képet betölti
}

```


<details>
<summary>Nyisd le az xaml forrásért</summary>
```c#
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media.Imaging;

namespace akasztofa
{
    public partial class MainWindow : Window
    {
        private Random rnd = new Random();
        private string word;
        private List<char> guessedLetters;
        private int attemptsLeft = 11;
        private List<char> guessedChars = new List<char>();
        private List<Button> wordButtons = new List<Button>();

        public MainWindow()
        {
            InitializeComponent();
            LoadWords();
            StartNewGame();
        }

        private void LoadWords()
        {
            try
            {
                string[] words = File.ReadAllLines("szavak.txt");

                words = words.Where(w => w.Length <= 20 && !w.Contains(" ") && !w.Contains("-")).ToArray();

                if (words.Length == 0)
                {
                    MessageBox.Show("A szavak.txt fájlban nincs megfelelő szó.");
                    Application.Current.Shutdown();
                }

                word = words[rnd.Next(words.Length)].ToUpper(); 
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Hiba történt a szavak betöltésekor: {ex.Message}");
                Application.Current.Shutdown();
            }
        }


        private void StartNewGame()
        {
            LoadWords(); 
            guessedLetters = new List<char>(new string('_', word.Length).ToCharArray());
            attemptsLeft = 11;
            guessedChars.Clear();
            CreateWordButtons();
            UpdateInfoText();
            UpdateHangmanImage();
            CreateLetterButtons();
        }


        private void CreateWordButtons()
        {
            WordDisplay.Children.Clear(); 
            wordButtons.Clear();

            for (int i = 0; i < word.Length; i++)
            {
                Button btn = new Button
                {
                    Content = "_", 
                    Tag = word[i].ToString(),
                    Name = "LetterButton" + i,
                    FontSize = 16,
                    IsEnabled = false 
                };
                wordButtons.Add(btn);
                
                WordDisplay.Children.Add(btn);
            }
        }

        private void UpdateInfoText()
        {
            InfoText.Text = $"Hibás próbálkozások: {11 - attemptsLeft}";
        }

        private void UpdateHangmanImage()
        {
            int errorCounter = 11 - attemptsLeft;
            HangmanImage.Source = new BitmapImage(new Uri($"Images/akasztofa{errorCounter}.png", UriKind.Relative));

        }

        private void CreateLetterButtons()
        {
            LetterButtons.Children.Clear();

            // Helyes magyar ABC sorrend
            string magyarABC = "AÁBCDEÉFGHIÍJKLMNOÓÖŐPQRSTUÚÜŰVWXYZ";

            foreach (char c in magyarABC)
            {
                Button button = new Button
                {
                    Content = c.ToString(),
                    FontSize = 16,
                    Width = 30,
                    Height = 30,
                    Margin = new Thickness(2),
                    IsEnabled = true
                };
                button.Click += LetterButton_Click;
                LetterButtons.Children.Add(button);
            }
        }


        private void LetterButton_Click(object sender, RoutedEventArgs e)
        {
            Button button = (Button)sender;
            char guessedChar = button.Content.ToString()[0];
            button.IsEnabled = false; 

            if (word.Contains(guessedChar))
            {
               
                for (int i = 0; i < word.Length; i++)
                {
                    if (word[i] == guessedChar)
                    {
                        wordButtons[i].Content = guessedChar.ToString(); 
                    }
                }
            }
            else
            {
                attemptsLeft--; 
            }

            UpdateInfoText();
            UpdateHangmanImage();

           
            if (attemptsLeft == 0)
            {
                MessageBox.Show($"Vesztettél! A szó a következő volt: {word}");
                StartNewGame();
            }
            else if (!wordButtons.Any(btn => btn.Content.ToString() == "_")) 
            {
                MessageBox.Show("Gratulálok, nyertél!");
                StartNewGame();
            }
        }
    }
}
```
```c#
```
</details>


<details>

<summary>Nyisd le a xaml.cs forrásért </summary>
```c#
<Window x:Class="akasztofa.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:akasztofa"
        mc:Ignorable="d"
        Title="Akasztófa" Height="450" Width="800">

    <!-- Fő Grid -->
    <Grid Name="MainGrid">
        <Grid.RowDefinitions>
            <RowDefinition Height="*" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*" />
            <ColumnDefinition Width="*" />
        </Grid.ColumnDefinitions>

        
        <Grid Name="PuzzleGrid" Grid.Row="0" Grid.Column="0" Background="LightGray">
            <StackPanel HorizontalAlignment="Center" VerticalAlignment="Center">
                <TextBlock Text="Akasztófa" FontSize="20" FontWeight="Bold" HorizontalAlignment="Center"/>
                <WrapPanel Name="WordDisplay" HorizontalAlignment="Center"/>
            </StackPanel>
        </Grid>
      
        <Grid Name="LetterGrid" Grid.Row="1" Grid.Column="0" Background="LightGray">
            <UniformGrid Name="LetterButtons" Columns="9" Rows="4" Margin="10">
                <!-- buttons -->
               
            </UniformGrid>
        </Grid>
       
        <Grid Name="InfoGrid" Grid.Row="0" Grid.Column="1" Background="LightGray">
            <TextBlock Name="InfoText" Text="Hibás próbálkozások: 0" FontSize="16" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </Grid>

        <Grid Name="HangmanGrid" Grid.Row="1" Grid.Column="1" Background="Transparent">
            <Image Name="HangmanImage" Width="157" Height="81" HorizontalAlignment="Center" VerticalAlignment="Center"/>
        </Grid>
    </Grid>
</Window>
```
</details>
