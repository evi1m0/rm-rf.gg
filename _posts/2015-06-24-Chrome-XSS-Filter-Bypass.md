---
layout: post
title: Chrome XSS Filter Bypass - 1187843005
---
- Discovery Date: 16-06-15
- Affected Products: Google Chrome 43.0.2357.124 m (letest stable version)
- Issue: [XSSAuditor: Dont give a pass to subsequent script blocks if first one is empty.](https://codereview.chromium.org/1187843005/)
- Description: Script bodies are filtered in chunks with the earliest chunk controlling the suppression of subsequent ones.  But we may fail to match if the first chunk can be reduced to an empty string.  Keep looking in that case.

/Source/core/html/parser/XSSAuditor.cpp:

    Old:

    m_state != SuppressingAdjacentCharacterTokens
    if ((m_state == SuppressingAdjacentCharacterTokens)
        || (m_scriptTagFoundInRequest && isContainedInRequest(canonicalizedSnippetForJavaScript(request)))) {
        request.token.eraseCharacters();
        request.token.appendToCharacter(' ');
        m_state = SuppressingAdjacentCharacterTokens;
        return true;
    }
    m_state = PermittingAdjacentCharacterTokens;

    ===================

    Fix:
    if (m_state == FilteringTokens && m_scriptTagFoundInRequest) {
        String snippet = canonicalizedSnippetForJavaScript(request);
        if (isContainedInRequest(snippet))
            m_state = SuppressingAdjacentCharacterTokens;
        else if (!snippet.isEmpty())
            m_state = PermittingAdjacentCharacterTokens;
    }
    if (m_state == SuppressingAdjacentCharacterTokens) {
        request.token.eraseCharacters();
        request.token.appendToCharacter(' ');
        return true;
    }
    
/Source/core/html/parser/XSSAuditor.h:

    68  private:
    69      static const size_t kMaximumFragmentLengthTarget = 100;
    70  
    71      enum State {
    72          Uninitialized,
    73          FilteringTokens,
    74          PermittingAdjacentCharacterTokens,
    75          SuppressingAdjacentCharacterTokens
    76      };
    77  
    78      enum TruncationKind {
    79          NoTruncation,
    80          NormalAttributeTruncation,
    81          SrcLikeAttributeTruncation,
    82          ScriptLikeAttributeTruncation
    83      };
    84  
    85      enum HrefRestriction {
    86          ProhibitSameOriginHref,
    87          AllowSameOriginHref
    88      };
    
Demo: 
    
    https://localhost/<svg><script>/<*/>alert()</script>
    
有趣的逻辑漏洞，花了点儿时间追下：

    if ((m_state == SuppressingAdjacentCharacterTokens)
    || (m_scriptTagFoundInRequest && isContainedInRequest(canonicalizedSnippetForJavaScript(request))))
    
条件：```if((a) || (b && c)) ```

其中 ```a || b``` 某一值为 false，则不会给 m_state 赋值 SACT 状态，跟踪变量 m_state 不为 SACT

使条件达成  ```if (false || (false && true))``` 绕过过滤。