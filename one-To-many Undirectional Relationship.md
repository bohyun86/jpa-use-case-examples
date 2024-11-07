### `Tournament`와 `Registration` 간의 단방향 `OneToMany` 관계를 설정하였습니다.

1. **엔티티 설명**:
    - `Tournament` 엔티티는 여러 개의 `Registration`을 가질 수 있습니다.
    - `@OneToMany`와 `@JoinColumn` 어노테이션을 사용하여 `tournament_id`를 외래 키로 설정하였습니다.

2. **단방향 관계**:
    - `Tournament` 엔티티에서만 `Registration` 리스트를 관리합니다. 즉, `Registration`은 `Tournament`를 참조하지 않습니다.

3. **리포지토리 및 서비스 생성**:
    - `TournamentService`와 `TournamentController`를 통해 `Tournament`를 CRUD하고 `Registration`을 추가할 수 있습니다.

``` java
package io.datajek.databaserelationships.onetomany.uni;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Tournament {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String location;

    @OneToMany(cascade = CascadeType.ALL) // Tournament 엔티티에서 발생한 변경 사항이 관련된 Registration 엔티티들에도 전파됩니다.
    @JoinColumn(name = "tournament_id", referencedColumnName = "id") // tournament_id는 데이터베이스의 외래 키 컬럼 이름이고, referencedColumnName = "id" 는 참조하는 엔티티(Tournament)의 기본 키 컬럼을 나타냅니다.
    private List<Registration> registrations = new ArrayList<>();

    // 기본 생성자
    public Tournament() {
    }

    // 생성자
    public Tournament(String name, String location) {
        this.name = name;
        this.location = location;
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

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }

    public List<Registration> getRegistrations() {
        return registrations;
    }

    public void setRegistrations(List<Registration> registrations) {
        this.registrations = registrations;
    }

    // cascade = CascadeType.ALL) 설정으로 인해 Registration을 추가할 때 Tournament도 함께 저장됩니다.
    // Registration을 Tournament에 추가합니다.
    public void addRegistration(Registration registration) {
        this.registrations.add(registration);
    }

    @Override
    public String toString() {
        return "Tournament{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", location='" + location + '\'' +
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

    @Override
    public String toString() {
        return "Registration{" +
                "id=" + id +
                ", matchType='" + matchType + '\'' +
                '}';
    }
}

@Repository
public interface TournamentRepository extends JpaRepository<Tournament, Long> {
}

@Repository
public interface RegistrationRepository extends JpaRepository<Registration, Long> {
}

@Service
public class TournamentService {
    @Autowired
    private TournamentRepository tournamentRepository;

    public Tournament saveTournament(Tournament tournament) {
        return tournamentRepository.save(tournament);
    }

    public Tournament getTournament(Long id) {
        return tournamentRepository.findById(id).orElse(null);
    }

    public void addRegistration(Long tournamentId, Registration registration) {
        Tournament tournament = getTournament(tournamentId);
        if (tournament != null) {
            tournament.addRegistration(registration);
            tournamentRepository.save(tournament);
        }
    }
}

@RestController
@RequestMapping("/tournaments")
public class TournamentController {
    @Autowired
    private TournamentService tournamentService;

    @PostMapping
    public Tournament createTournament(@RequestBody Tournament tournament) {
        return tournamentService.saveTournament(tournament);
    }

    @GetMapping("/{id}")
    public Tournament getTournament(@PathVariable Long id) {
        return tournamentService.getTournament(id);
    }

    @PutMapping("/{id}/registrations")
    public Tournament addRegistration(@PathVariable Long id, @RequestBody Registration registration) {
        tournamentService.addRegistration(id, registration);
        return tournamentService.getTournament(id);
    }
}



```