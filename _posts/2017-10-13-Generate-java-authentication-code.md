---
layout: post
date: 2017-10-10 11:00
title: Java 验证码
description: 总结下之前写的Java验证码实现和注意点。
categories: [Java]
tags: [Java,验证码]
---

# 验证码生成工具类代码
``` java

import java.awt.Color;
import java.awt.Font;
import java.awt.Graphics2D;
import java.awt.image.BufferedImage;
import java.util.Random;

/**
 * 生成验证码
 * 
 * @author yanlz
 * @date 2017/10/13
 */
public class SecurityCodeUtil {

	private static int width = 100;

	private static int height = 40;

	private static int codeCount = 4;

	private static int fontWidth = width / (codeCount + 1);
	private static int fontHeight = 24;
	private static int codeY = 28;

	public static final char[] codeSequence = { 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'k', 'm', 'n', 'p', 'q', 'r',
			's', 't', 'u', 'v', 'w', 'x', 'y', 'z', '2', '3', '4', '5', '6', '7', '8', '9' };

	public static String generateSecurityCode(BufferedImage imgBuf) {
		Graphics2D g = imgBuf.createGraphics();

		// 随机图片生成器
		Random random = new Random();

		// 图片背景为白色
		g.setColor(Color.WHITE);
		g.fillRect(0, 0, width, imgBuf.getHeight());

		// 创建和设值字体，字体的大小根据图片的高度来定
		Font font = new Font("Fixedsys", Font.BOLD, fontHeight);
		g.setFont(font);

		// 画边框
		g.setColor(Color.BLACK);
		g.drawRect(0, 0, width - 1, height - 1);

		// 随机产生160条干扰线
		g.setColor(Color.LIGHT_GRAY);
		for (int i = 0; i < 160; i++) {
			int x = random.nextInt(width);
			int y = random.nextInt(height);
			int xl = random.nextInt(12);
			int yl = random.nextInt(12);
			g.drawLine(x, y, x + xl, y + yl);
		}

		// randomCode用于保存随机产生的验证码以便用户登录后进行验证。
		StringBuffer randomCode = new StringBuffer();
		int red = 0, green = 0, blue = 0;

		// 随机产生codeCount数字的验证码。
		for (int i = 0; i < codeCount; i++) {
			// 得到随机产生的验证码数字。
			String strRand = String.valueOf(codeSequence[random.nextInt(30)]);
			// 产生随机的颜色分量来构造颜色值，这样输出的每位数字的颜色值都将不同。
			red = random.nextInt(70);
			green = random.nextInt(70);
			blue = random.nextInt(70);

			// 用随机产生的颜色将验证码绘制到图像中。
			g.setColor(new Color(red, green, blue));
			g.drawString(strRand, (i + 1) * (fontWidth + 2) - 12, codeY);

			// 将产生的四个随机数组合在一起。
			randomCode.append(strRand);
		}
		return randomCode.toString();
	}

	/**
	 *
	 * 初始化一个图片缓存区。
	 * 
	 * @return
	 */
	public static BufferedImage initImage() {
		BufferedImage buffImg = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);

		return buffImg;
	}
}
```
# 登录控制层代码
## 登录主方法
```java
 /**
     * 用户登录方法
     *
     * @param loginParam 登录相关参数
     * @param request    请求实体
     * @return
     */
    @PostMapping(value = "/login")
    public ResponseModel login(HttpServletRequest request, HttpServletResponse response,
                               @RequestBody LoginParam loginParam) {
    // 检查验证码是否输入
        if (StringUtils.isEmpty(loginParam.getSecurityCode())) {
            removeSecurityCode(response, request); // 销毁该验证码，重新生成
            return new ResponseModel(HttpStatus.UNAUTHORIZED, "验证码为空，请重新输入。");
        }

        // 检查验证码是否正确
        if (!checkSecurityCode(loginParam.getSecurityCode(), request)) {
            removeSecurityCode(response, request); // 销毁该验证码，重新生成
            return new ResponseModel(HttpStatus.UNAUTHORIZED, "验证码错误，请重新输入。");
        }
        
        ......
    
    }
    
    /**
     * 校验验证码
     *
     * @param securityCode 验证码
     */
    private Boolean checkSecurityCode(String securityCode, HttpServletRequest request) {
        if (securityCode == null || securityCode.isEmpty() || request == null) {
            return false;
        }
        return (securityCode.toLowerCase()).equals(request.getSession().getAttribute(SECURITY_CODE_SESSION));
    }
    
    /**
     * 显示验证码图片
     *
     * @param response HttpServletResponse
     */
    @GetMapping(value = "/security_code")
    public void getSecurityCode(HttpServletResponse response, HttpServletRequest request) {
        BufferedImage imgBuf = SecurityCodeUtil.initImage();

        String securityCode = SecurityCodeUtil.generateSecurityCode(imgBuf);

        setSecurityCode(securityCode, request);

        response.setContentType("image/jpeg");
        response.setHeader("Pragma", "no-cache");
        response.setHeader("Cache-Control", "no-cache");
        response.setDateHeader("Expires", 0);

        // 将图像输出到Servlet输出流中。
        ServletOutputStream sos = null;
        try {
            sos = response.getOutputStream();
            ImageIO.write(imgBuf, "jpeg", sos);
        } catch (IOException e) {
            logger.error("生成验证码时错误", e);
        } finally {
            try {
                sos.close();
            } catch (IOException e) {
                logger.error("生成验证码时错误", e);
            }
        }
    }
    
    
    /**
     * 设置验证码
     *
     * @param securityCode
     * @param request
     */
    private void setSecurityCode(String securityCode, HttpServletRequest request) {
        // 验证码存储在session，5分钟内有效
        HttpSession session = request.getSession();
        session.setAttribute(SECURITY_CODE_SESSION, securityCode);
    }
    
    /**
     * 删除验证码
     *
     * @param request
     * @param response
     */
    private void removeSecurityCode(HttpServletResponse response, HttpServletRequest request) {
        // 用户登录成功后删除session中存放的验证码
        request.getSession().removeAttribute(SECURITY_CODE_SESSION);
        // 从Cookie中删除验证码令牌
        Cookie cookie = new Cookie(COOKIE_SECURITY_CODE, null);
        cookie.setDomain(domainName);
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
    
    
```

## 用户登录实体
``` java
public class LoginParam {

	private String username;

	private String password;

	private String securityCode;

	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public String getSecurityCode() {
		return securityCode;
	}

	public void setSecurityCode(String securityCode) {
		this.securityCode = securityCode;
	}
}
```

# 系统过滤器放行验证码接口请求
```java
@WebFilter(urlPatterns = { "/api/*" })
public class LoginFilter implements Filter {

	private final Logger logger = LoggerFactory.getLogger(getClass());

	@Value("${access.allow.origin}")
	private String accessAllowOrigin;

	private Set<String> excludeUrls = new HashSet<>();

	@Override
	public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) resp;
		HttpSession session = request.getSession(false);

		request.setCharacterEncoding("UTF-8");
		response.setCharacterEncoding("UTF-8");

		if (accessAllowOrigin != null) {
			response.addHeader("Access-Control-Allow-Origin", this.getProtocol(request) + accessAllowOrigin);
		} else {
			response.addHeader("Access-Control-Allow-Origin", "*");
		}
		response.addHeader("Access-Control-Allow-Credentials", "true");
		response.addHeader("Access-Control-Allow-Headers",
				"Origin, X-Requested-With, Content-Type, Accept,X-Pagination");
		response.addHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS");
		response.addHeader("Access-Control-Expose-Headers", "X-Pagination");

		// 解决跨域情况下 浏览器请求OPTIONS方法失败的问题
		if (request.getMethod().equals("OPTIONS")) {
			return;
		}

		if (isInclude(request.getServletPath())) {
			if (session != null) {
				LoginEntity loginEntity = (LoginEntity) session.getAttribute(UserContext.LOGINENTITY);
				if (loginEntity != null) {
					UserContext.set(loginEntity);
					chain.doFilter(request, response);
					return;
				}
			}
			unLogin(response);
		} else {
			chain.doFilter(request, response);
		}
	}

	/**
	 * 判断url是否需要身份认证
	 * 
	 * @param servletPath
	 * @return
	 */
	private boolean isInclude(String servletPath) {
		if (excludeUrls.contains(servletPath)) {
			return false;
		}
		return true;
	}

	/**
	 * 用于提示用户未登录
	 * 
	 * @param response
	 */
	private void unLogin(HttpServletResponse response) {
		PrintWriter out = null;
		try {
			out = response.getWriter();
			response.setContentType("application/json");
			JSONObject result = new JSONObject();
			result.put("status", 401);
			result.put("error", "用户未登录");
			out.print(result.toJSONString());
		} catch (IOException e) {
			logger.error("Filter异常！", e);
		} finally {
			if (out != null) {
				out.close();
			}
		}
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		excludeUrls.add("/api/login");
		excludeUrls.add("/api/logout");
		excludeUrls.add("/api/security_code");
	}

	@Override
	public void destroy() {

	}

	/**
	 * 根据HttpServletRequest获取请求协议
	 * 
	 * @param request
	 * @return http:// 或 https://
	 */
	private String getProtocol(HttpServletRequest request) {
		String protocol = request.getHeader("X-Forwarded-Proto");
		if (protocol == null || protocol.length() == 0 || "unknown".equalsIgnoreCase(protocol)) {
			protocol = "http";
		}
		return protocol + "://";
	}
}
```




