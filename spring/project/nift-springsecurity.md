# Spring Security 필터에서 JWT 인증 처리 개선하기

SpringSecurity를 사용하면서, JWT 토큰을 인증하는 부분에 있어서 비효율적으로 진행되는 것 같아 추가적으로 공부 진행\
//TODO: 개념도 업로드 할 예정

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws IOException {
    try {
        String token = resolveToken(request);
        if (token != null) {
            if (!jwtTokenProvider.validateToken(token)) {
                throw new IllegalArgumentException("유효하지 않은 토큰입니다.");
            }

            Long userId = jwtTokenProvider.getUserFromToken(token).getUserId();
            UserDetails userDetails = userDetailsService.loadUserByUsername(userId.toString());

            if (userDetails instanceof CustomUserDetails customUserDetails) {
                Authentication authentication =
                        new UsernamePasswordAuthenticationToken(customUserDetails, null, customUserDetails.getAuthorities());

                SecurityContextHolder.getContext().setAuthentication(authentication);
            } else {
                log.error("userDetails is not CustomUserDetails. Authentication not set.");
            }
        }
        filterChain.doFilter(request, response);

    } catch (UsernameNotFoundException | IllegalArgumentException e) {
        log.warn("Authentication failed: {}", e.getMessage());
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().write("{\"message\": \"Invalid or expired token.\"}");
    } catch (Exception e) {
        log.error("Cannot set user authentication", e);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().write("{\"message\": \"Authentication error.\"}");
    }
}
```

```java
@PostMapping("/{articleId}/purchase")
public ResponseEntity<?> createContract(
        @AuthenticationPrincipal CustomUserDetails userDetails,
        @Parameter(name = "articleId", description = "거래할 게시글 ID", example = "1")
        @PathVariable("articleId") Long articleId
) {
    contractService.addArticleHistory(articleId, userDetails.getUserId());

    return ResponseEntity.noContent().build();
}
```
