### `Player`와 `Registration` 간의 양방향 `OneToMany` 및 `ManyToOne` 관계를 설정했습니다.

### 주요 변경 사항 요약:
1. **양방향 관계 설정**:
    - `Player` 클래스에 `List<Registration>` 필드를 추가하고 `@OneToMany` 어노테이션을 사용했습니다. 이때 `mappedBy` 속성을 사용하여 `Registration` 클래스에서 이 관계를 소유하도록 지정했습니다.
    - `Registration` 클래스에 `Player` 필드를 추가하고 `@ManyToOne` 및 `@JoinColumn` 어노테이션을 사용하여 외래 키(`player_id`)로 참조하도록 설정했습니다.

2. **양방향 연결 메서드**:
    - `Player` 클래스에 `addRegistration()` 메서드를 추가하여 `Registration` 객체를 등록하고, 해당 `Registration` 객체에도 연관된 `Player`를 설정하여 관계를 양방향으로 유지했습니다.

3. **서비스 및 컨트롤러 수정**:
    - `PlayerService` 클래스의 `addRegistration()` 메서드를 수정하여 `Player`와 `Registration`의 양방향 관계를 적절히 반영하도록 했습니다.
    - `PlayerController` 클래스에서 등록을 추가하는 `PUT` 매핑을 제공하여 클라이언트에서 등록을 쉽게 추가할 수 있도록 했습니다.

이 코드로 `Player`와 `Registration` 간의 양방향 관계를 설정하고 데이터베이스에서 이 두 엔티티를 적절히 연결할 수 있습니다. 

```java
package io.datajek.databaserelationships.onetomany.bi;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Player {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "player", cascade = CascadeType.ALL) // Registration 엔티티에서 관계를 소유하므로 mappedBy 속성을 사용합니다. Player 엔티티에서 발생한 변경 사항이 관련된 Registration 엔티티들에도 전파됩니다.
    private List<Registration> registrations = new ArrayList<>(); 

    // 기본 생성자
    public Player() {
    }

    // 생성자
    public Player(String name) {
        this.name = name;
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

    public List<Registration> getRegistrations() {
        return registrations;
    }

    public void setRegistrations(List<Registration> registrations) {
        this.registrations = registrations;
    }

    // cascade = CascadeType.ALL 설정으로 인해 Registration을 추가할 때 Player도 함께 저장됩니다.
    // Registration을 Player에 추가하고, Registration에도 Player를 설정하여 양방향 관계를 설정합니다.
    public void addRegistration(Registration registration) {
        this.registrations.add(registration);
        registration.setPlayer(this);
    }

    @Override
    public String toString() {
        return "Player{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", registrations=" + registrations +
                '}';
    }
}

@Entity
public class Registration {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String matchType;

    @ManyToOne
    @JoinColumn(name = "player_id", referencedColumnName = "id")
    private Player player;

    // 기본 생성자
    public Registration() {
    }

    // 생성자
    public Registration(String matchType) {
        this.matchType = matchType;
    }

    // Getter & Setter
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getMatchType() {
        return matchType;
    }

    public void setMatchType(String matchType) {
        this.matchType = matchType;
    }

    public Player getPlayer() {
        return player;
    }

    public void setPlayer(Player player) {
        this.player = player;
    }

    @Override
    public String toString() {
        return "Registration{" +
                "id=" + id +
                ", matchType='" + matchType + '\'' +
                ", player=" + (player != null ? player.getName() : "null") +
                '}';
    }
}

@Repository
public interface PlayerRepository extends JpaRepository<Player, Long> {
}

@Repository
public interface RegistrationRepository extends JpaRepository<Registration, Long> {
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

    public void addRegistration(Long playerId, Registration registration) {
        Player player = getPlayer(playerId);
        if (player != null) {
            player.addRegistration(registration);
            playerRepository.save(player);
        }
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

    @PutMapping("/{id}/registrations")
    public Player addRegistration(@PathVariable Long id, @RequestBody Registration registration) {
        playerService.addRegistration(id, registration);
        return playerService.getPlayer(id);
    }
}
```