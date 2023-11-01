struktura vypda nejak takto
- main
- -projekt
  - -config
  - -controllers
  - -dtos
  - -entities
  - -enums
  - -interfaces
  - -services
  - -util
  - -.... atd
  - -resources
- test
- ....
____________________________________________

ukazka z controlleru

@RestController
@RequestMapping("/user")
@AllArgsConstructor
public class UserDetailController {

    private final UserDetailServiceImpl userDetailService;
	
	@PostMapping("/register")
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<ResponseEntity<UserDetailResponseDto>> create(@Validated @RequestBody Mono<UserDetailRequestDto> user, ServerWebExchange exchange) {
        return userDetailService.create(user, exchange.getRequest()).map(dto -> ResponseEntity.status(HttpStatus.CREATED).body(dto))
                .onErrorResume(error -> Mono.error(new ResponseStatusException(HttpStatus.BAD_REQUEST,
                        "Failed to create user with reason -> " + error.getMessage())));
    }
	
	.....
_______________________________________________
	
ukazka z dto

- @Data
- @ToString
- @AllArgsConstructor
- @Builder
- @NoArgsConstructor(force = true)
- public class UserDetailRequestDto {
    - private Integer id;
    - @NonNull
    - @Pattern(regexp = "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$", message = "Invalid email address")
    - private String email;
    - @NonNull
    - @Size(min = 5, message = "Password must have at least 5 characters")
    - private String password;
    - @NonNull
    - private String language;
    - @NonNull
    - private String version;
- }

- @Data
- @ToString
- @AllArgsConstructor
- @NoArgsConstructor(force = true)
- public class UserDetailResponseDto {
    - @NonNull
    - private Integer id;
    - @NonNull
    - private String email;
    - private LocalDateTime createdAt;
    - private LocalDateTime modifiedAt;
    - @NonNull
    - private Integer activeEmails;
    - @NonNull
    - private String language;
    - @NonNull
    - private Integer languageId;
    - private PaymentResponseDto payment;
    - private String version;
- }
__________________________________________________

ukazka entity

- @Data
- @ToString
- @AllArgsConstructor
- @NoArgsConstructor(force = true)
- @EqualsAndHashCode(callSuper = true)
- @Table("user_detail")
- public class UserDetailEntity extends BaseEntity {
    - @Id
    - private Integer id;
    - @NonNull
    - private String email;
    - @NonNull
    - private String password;
    - @NonNull
    - @Column("is_active")
    - private Boolean isActive;
   - ....
    - @Column("active_emails")
    - private Integer activeEmails;
  - .....
    - @Column("language_id")
    - private Integer languageId;
    - @Column("currency_id")
    - private Integer currencyId;
    - @Column("version_id")
    - private Integer versionId;
    - @Column("show_message")
    - private Boolean showMessage;
- }

_______________________________________________

ukazka service

@Service
@RequiredArgsConstructor
public class UserDetailServiceImpl implements ReactiveUserDetailsService {

    private final UserDetailRepository userDetailRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder;
    private final JwtTokenProvider jwtTokenProvider;
    private final TokenRevocationHandler tokenRevocationHandler;
    private final EmailService emailService;
    private final PaymentsRepository paymentsRepository;
    private final CurrencyRepository currencyRepository;
    private final LanguageRepository languageRepository;
    private final BankAccountService bankAccountService;
    private final VersionRepository versionRepository;
	
	napr.metoda pro update user details
	
	public Mono<UserDetailResponseDto> update(int id, Mono<UserDetailRequestDto> userDetailRequestDtoMono) {
        return userDetailRepository.findByDeletedAtIsNullAndId(id)
                .switchIfEmpty(Mono.error(new ResponseStatusException(HttpStatus.NOT_FOUND, "User not found")))
                .flatMap(entity ->
                        userDetailRequestDtoMono.flatMap(dto -> {
                            if (dto.getId() != id) {
                                return Mono.error(new ResponseStatusException(HttpStatus.BAD_REQUEST, "Cannot update user"));
                            }
                            String hashedPassword = bCryptPasswordEncoder.encode(dto.getPassword());
                            dto.setPassword(hashedPassword);
                            UserDetailEntity updatedUserDetailEntity = MapperEntityDto.toEntity(entity, dto, UserDetailEntity.class, OperationType.UPDATE);
                            return userDetailRepository.save(updatedUserDetailEntity).map(userDetailEntity -> MapperEntityDto.toDto(userDetailEntity, UserDetailResponseDto.class));
                        }));
    }
	
	.....
	
	
 ___________________________________________
	
- struktura je klasicka pro vetsinu spring-boot aplikaci :)
