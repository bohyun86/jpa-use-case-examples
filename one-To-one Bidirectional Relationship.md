### 요약
- `PlayerProfile` 클래스에서 `Player`를 참조할 수 있습니다. 
- `@JsonManagedReference`와 `@JsonBackReference`를 사용하여 JSON 직렬화 시 무한 재귀 오류를 방지했습니다. 
- 이 수정으로 `Player`와 `PlayerProfile` 간의 양방향 관계를 처리할 수 있습니다. 


``` java 
package io.datajek.databaserelationships.onetoone;

import com.fasterxml.jackson.annotation.JsonBackReference;
import com.fasterxml.jackson.annotation.JsonManagedReference;
import javax.persistence.*;

@Entity
public class Player {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToOne(cascade = CascadeType.ALL) // Player 엔티티에서 발생한 변경 사항이 관련된 PlayerProfile 엔티티에도 전파됩니다.
    @JoinColumn(name = "profile_id", referencedColumnName = "id") // profile_id는 데이터베이스의 외래 키 컬럼 이름이고, referencedColumnName = "id" 는 참조하는 엔티티(PlayerProfile)의 기본 키 컬럼을 나타냅니다.
    @JsonManagedReference
    private PlayerProfile playerProfile;

    // 기본 생성자
    public Player() {
    }

    // 생성자
    public Player(String name, PlayerProfile playerProfile) {
        this.name = name;
        this.playerProfile = playerProfile;
    }

    // Getter & Setter
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public PlayerProfile getPlayerProfile() {
        return playerProfile;
    }

    public void setPlayerProfile(PlayerProfile playerProfile) {
        this.playerProfile = playerProfile;
    }

    @Override
    public String toString() {
        return "Player{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", playerProfile=" + playerProfile +
                '}';
    }
}

@Entity
public class PlayerProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String twitterHandle;

    @OneToOne(mappedBy = "playerProfile")
    @JsonBackReference
    private Player player;

    // 기본 생성자
    public PlayerProfile() {
    }

    // 생성자
    public PlayerProfile(String twitterHandle) {
        this.twitterHandle = twitterHandle;
    }

    // Getter & Setter
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTwitterHandle() {
        return twitterHandle;
    }

    public void setTwitterHandle(String twitterHandle) {
        this.twitterHandle = twitterHandle;
    }

    public Player getPlayer() {
        return player;
    }

    public void setPlayer(Player player) {
        this.player = player;
    }

    @Override
    public String toString() {
        return "PlayerProfile{" +
                "id=" + id +
                ", twitterHandle='" + twitterHandle + '\'' +
                ", player=" + (player != null ? player.getName() : "null") +
                '}';
    }
}

@Repository
public interface PlayerRepository extends JpaRepository<Player, Long> {
}

@Repository
public interface PlayerProfileRepository extends JpaRepository<PlayerProfile, Long> {
}

@Service
public class PlayerService {
    @Autowired
    private PlayerRepository playerRepository;

    public Player savePlayer(Player player) {
        return playerRepository.save(player);
    }

    public Player getPlayer(Long id) {
        return playerRepository.findById(id).orElse(null);
    }
}

@RestController
@RequestMapping("/players")
public class PlayerController {
    @Autowired
    private PlayerService playerService;

    @PostMapping
    public Player createPlayer(@RequestBody Player player) {
        return playerService.savePlayer(player);
    }

    @GetMapping("/{id}")
    public Player getPlayer(@PathVariable Long id) {
        return playerService.getPlayer(id);
    }
}



```