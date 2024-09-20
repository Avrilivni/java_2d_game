```java
// MainActivity.java
package com.example.aretzir;

import androidx.appcompat.app.AppCompatActivity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.LinearLayout;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    private EditText usernameInput;
    private Button loginButton;
    private LinearLayout loginForm;
    private LinearLayout gameOptions;
    private Button playRandomButton;
    private Button playWithFriendButton;
    private LinearLayout friendInput;
    private EditText friendUsernameInput;
    private Button startGameWithFriendButton;

    private String currentUser;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initialize views
        usernameInput = findViewById(R.id.username_input);
        loginButton = findViewById(R.id.login_button);
        loginForm = findViewById(R.id.login_form);
        gameOptions = findViewById(R.id.game_options);
        playRandomButton = findViewById(R.id.play_random_button);
        playWithFriendButton = findViewById(R.id.play_with_friend_button);
        friendInput = findViewById(R.id.friend_input);
        friendUsernameInput = findViewById(R.id.friend_username_input);
        startGameWithFriendButton = findViewById(R.id.start_game_with_friend_button);

        // Set click listeners
        loginButton.setOnClickListener(v -> login());
        playRandomButton.setOnClickListener(v -> playRandom());
        playWithFriendButton.setOnClickListener(v -> showFriendInput());
        startGameWithFriendButton.setOnClickListener(v -> playWithFriend());
    }

    private void login() {
        String username = usernameInput.getText().toString().trim();
        if (!username.isEmpty()) {
            currentUser = username;
            loginForm.setVisibility(View.GONE);
            gameOptions.setVisibility(View.VISIBLE);
        } else {
            Toast.makeText(this, "נא להזין שם משתמש", Toast.LENGTH_SHORT).show();
        }
    }

    private void playRandom() {
        Toast.makeText(this, "מחפש שחקן אקראי...", Toast.LENGTH_SHORT).show();
        startGame(currentUser, "שחקן אקראי");
    }

    private void showFriendInput() {
        friendInput.setVisibility(View.VISIBLE);
    }

    private void playWithFriend() {
        String friendUsername = friendUsernameInput.getText().toString().trim();
        if (!friendUsername.isEmpty()) {
            Toast.makeText(this, "מתחבר למשחק עם " + friendUsername, Toast.LENGTH_SHORT).show();
            startGame(currentUser, friendUsername);
        } else {
            Toast.makeText(this, "נא להזין את שם המשתמש של החבר", Toast.LENGTH_SHORT).show();
        }
    }

    private void startGame(String player1, String player2) {
        Intent intent = new Intent(this, GameActivity.class);
        intent.putExtra("players", new String[]{player1, player2});
        startActivity(intent);
    }
}

// GameActivity.java
package com.example.aretzir;

import androidx.appcompat.app.AppCompatActivity;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Random;

public class GameActivity extends AppCompatActivity {

    private TextView letterDisplay;
    private TextView playerNameDisplay;
    private EditText countryInput, cityInput, animalInput, plantInput, objectInput;
    private EditText girlNameInput, boyNameInput, professionInput;
    private Button submitButton;
    private Button newRoundButton;

    private List<Player> players;
    private int currentPlayerIndex = 0;
    private String currentLetter;
    private static final String HEBREW_LETTERS = "אבגדהוזחטיכלמנסעפצקרשת";
    private GameRound currentRound;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_game);

        // Initialize views
        letterDisplay = findViewById(R.id.letter_display);
        playerNameDisplay = findViewById(R.id.player_name_display);
        countryInput = findViewById(R.id.country_input);
        cityInput = findViewById(R.id.city_input);
        animalInput = findViewById(R.id.animal_input);
        plantInput = findViewById(R.id.plant_input);
        objectInput = findViewById(R.id.object_input);
        girlNameInput = findViewById(R.id.girl_name_input);
        boyNameInput = findViewById(R.id.boy_name_input);
        professionInput = findViewById(R.id.profession_input);
        submitButton = findViewById(R.id.submit_button);
        newRoundButton = findViewById(R.id.new_round_button);

        // Initialize players
        String[] playerNames = getIntent().getStringArrayExtra("players");
        players = new ArrayList<>();
        for (String name : playerNames) {
            players.add(new Player(name));
        }

        submitButton.setOnClickListener(v -> submitAnswers());
        newRoundButton.setOnClickListener(v -> startNewRound());

        startNewRound();
    }

    private void startNewRound() {
        chooseLetter();
        resetInputs();
        updatePlayerDisplay();
        submitButton.setVisibility(View.VISIBLE);
        newRoundButton.setVisibility(View.GONE);
        currentRound = new GameRound(currentLetter, players);
    }

    private void chooseLetter() {
        Random random = new Random();
        currentLetter = String.valueOf(HEBREW_LETTERS.charAt(random.nextInt(HEBREW_LETTERS.length())));
        letterDisplay.setText(currentLetter);
    }

    private void resetInputs() {
        countryInput.setText("");
        cityInput.setText("");
        animalInput.setText("");
        plantInput.setText("");
        objectInput.setText("");
        girlNameInput.setText("");
        boyNameInput.setText("");
        professionInput.setText("");
    }

    private void updatePlayerDisplay() {
        playerNameDisplay.setText("תור של: " + players.get(currentPlayerIndex).getName());
    }

    private void submitAnswers() {
        Player currentPlayer = players.get(currentPlayerIndex);
        currentRound.addAnswers(currentPlayer, 
            countryInput.getText().toString(),
            cityInput.getText().toString(),
            animalInput.getText().toString(),
            plantInput.getText().toString(),
            objectInput.getText().toString(),
            girlNameInput.getText().toString(),
            boyNameInput.getText().toString(),
            professionInput.getText().toString()
        );

        currentPlayerIndex++;
        
        if (currentPlayerIndex >= players.size()) {
            // Round finished
            currentRound.calculateScores();
            showResults();
        } else {
            updatePlayerDisplay();
            resetInputs();
        }
    }

    private void showResults() {
        Intent intent = new Intent(this, ResultsActivity.class);
        intent.putExtra("round", currentRound);
        startActivity(intent);
    }
}

// ResultsActivity.java
package com.example.aretzir;

import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;

public class ResultsActivity extends AppCompatActivity {

    private TextView resultsTextView;
    private Button nextRoundButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_results);

        resultsTextView = findViewById(R.id.results_text_view);
        nextRoundButton = findViewById(R.id.next_round_button);

        GameRound round = (GameRound) getIntent().getSerializableExtra("round");
        displayResults(round);

        nextRoundButton.setOnClickListener(v -> finish());
    }

    private void displayResults(GameRound round) {
        StringBuilder sb = new StringBuilder();
        sb.append("תוצאות הסיבוב:\n\n");

        for (Player player : round.getPlayers()) {
            sb.append(player.getName()).append(": ").append(player.getScore()).append(" נקודות\n");
        }

        resultsTextView.setText(sb.toString());
    }
}

// Player.java
package com.example.aretzir;

import java.io.Serializable;

public class Player implements Serializable {
    private String name;
    private int score;

    public Player(String name) {
        this.name = name;
        this.score = 0;
    }

    public String getName() {
        return name;
    }

    public int getScore() {
        return score;
    }

    public void addScore(int points) {
        score += points;
    }
}

// GameRound.java
package com.example.aretzir;

import java.io.Serializable;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class GameRound implements Serializable {
    private String letter;
    private List<Player> players;
    private Map<Player, Map<String, String>> answers;

    public GameRound(String letter, List<Player> players) {
        this.letter = letter;
        this.players = players;
        this.answers = new HashMap<>();
    }

    public void addAnswers(Player player, String country, String city, String animal, String plant,
                           String object, String girlName, String boyName, String profession) {
        Map<String, String> playerAnswers = new HashMap<>();
        playerAnswers.put("country", country);
        playerAnswers.put("city", city);
        playerAnswers.put("animal", animal);
        playerAnswers.put("plant", plant);
        playerAnswers.put("object", object);
        playerAnswers.put("girlName", girlName);
        playerAnswers.put("boyName", boyName);
        playerAnswers.put("profession", profession);
        answers.put(player, playerAnswers);
    }

    public void calculateScores() {
        for (String category : answers.get(players.get(0)).keySet()) {
            Map<String, Integer> answerCounts = new HashMap<>();
            for (Player player : players) {
                String answer = answers.get(player).get(category).toLowerCase();
                if (answer.startsWith(letter.toLowerCase())) {
                    answerCounts.put(answer, answerCounts.getOrDefault(answer, 0) + 1);
                }
            }

            for (Player player : players) {
                String answer = answers.get(player).get(category).toLowerCase();
                if (answer.startsWith(letter.toLowerCase())) {
                    if (answerCounts.get(answer) == 1) {
                        player.addScore(15);
