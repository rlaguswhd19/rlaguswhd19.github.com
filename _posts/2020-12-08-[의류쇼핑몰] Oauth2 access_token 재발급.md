---
title: "[의류쇼핑몰] 의류 쇼핑몰 로그인/로그아웃"
tags:
 - 의류쇼핑몰
 - project

---

# 의류 쇼핑몰 로그인/로그아웃

먼저 로그인과 로그아웃을 생각했다. 차근차근 다시 시작한다는 마인드로 로그인을 했을때 어떤 데이터를 가지고 있을까 생각해봤다.



로그인을 하게되면 access_token을 받게된다. 그리고 유저의 정보를 저장할게 필요하다. 예를들면 id나 username이 필요하다. 그이유는 이를 이용하여 account(계정) 정보를 사용할 필요가 있기 때문이다. 이왕이면 id를 가지고 있는게 깔끔할것 같다.

<br/>

#### account 계정정보를 활용한 이슈

> user : 배송조회, 장바구니
>
> admin : 옷 등록, 등록한 옷 리스트

지금까지 생각한건 이정도.. 아무튼 account정보를 가지고 있어야 한다.

그래서 로그인에 성공하면 username을 활용해 id를 가져와 session에 저장해놓겠다.

<br/>

<br/>

로그인한 정보가 30분 이후에 자동으로 로그아웃 되도록 한다. 

그러기 위해서는 옆에 sidebar를 만들어서 token의 유효시간을 보여주면서 refresh_token을 이용해서 연장할 수 있도록 구현하겠다. access_token의 expires_in은 30분으로 0이되면 자동으로 로그아웃이 되도록 sessions을 날려버린다. 문제는 중간에 로그아웃을 했을때 Spring에 어떤 요청을 보내 로그아웃을 시켜야 한다는 것이다.

<br/>

## 오늘 구현한 이슈

1. ##### Dress와 Account의 양방향관계 (되는지는 아직 잘 모르겠다.)

   Dress가 JoinColumn을 사용해 주인이고 Account가 읽을수만 있다. cascade설정을 해주지 않아서 Dress를 등록할때 확인할 예정이다.

```java
public class Account {
	// 양방향 드레스 리스트
	@OneToMany(mappedBy = "account")
	private Set<Dress> dress_arr = new HashSet<Dress>();
	}
}

public class Dress {	
	@ManyToOne
	@JoinColumn(name = "account_id")
	private Account account;
	}
}
```

<br/>

2. ##### Dpage삭제 및 Dress 추가(comment는 하지 않음)

   <br/>

3. ##### CorsConfiguration이 작동하지 않아서 예전에 했던 CorsFilter로 대체했다. (됐었는데 왜 안돼..)

   <br/>

4. ##### email을 통해서 Account의 id를 반환 (Test 및 front 구현)

   ```java
   @GetMapping("/{email}")
   public ResponseEntity<?> findByEmail(@PathVariable String email){
   	return accountService.findByEmail(email);
   }
   ```

   <br/>

5. ##### refresh_token을 활용해 access_token을 받음 (Test 및 front 구현)

```java
@Test
	@TestDescription("refresh_token을 활용하여 access_token 재발급 Test")
	public void getAccessTokenByRefreshToken() throws Exception {
		
		ResultActions perform = mockMvc.perform(post("/oauth/token")
				.with(httpBasic(appProperties.getClientId(), appProperties.getClientSecret()))
				.param("username", appProperties.getUserEmail())
				.param("password", appProperties.getUserPassword())
				.param("grant_type", "password")
				);
		
		var responseBody = perform.andReturn().getResponse().getContentAsString();
		Jackson2JsonParser parser = new Jackson2JsonParser();
		String reefresh_token = parser.parseMap(responseBody).get("refresh_token").toString();
		
		mockMvc.perform(post("/oauth/token")
				.with(httpBasic(appProperties.getClientId(), appProperties.getClientSecret()))
				.param("refresh_token", reefresh_token)
				.param("grant_type", "refresh_token")
				)
				.andDo(print())
				.andExpect(status().isOk())
				.andExpect(jsonPath("access_token").exists())
				.andExpect(jsonPath("expires_in").exists())
				;
	}
```