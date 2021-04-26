## 关于签名
java 签名算法: 理论上就是字典序排序 key 再根据 key 拼接。
```Java
	//签名
    public static void sign(Map<String, Object> params, String key) {
        String content = getSignCheckContent(params);
        content += "&projectKey=" + key;
        String sign = Md5Util.md5(content);
        params.put("sign", sign);
    }
    
	//排序拼接
    public static String getSignCheckContent(Map<String, Object> params) {
        if (params == null) {
            return null;
        }

        StringBuffer content = new StringBuffer();
        List<String> keys = new ArrayList<>(params.keySet());
				//字典序排序
        Collections.sort(keys);

				//根据 key 获得 value 拼接
        for (int i = 0; i < keys.size(); i++) {
            String key = keys.get(i);
            String value = String.valueOf(params.get(key));
            content.append((i == 0 ? "" : "&") + key + "=" + value);
        }

        return content.toString();
    }
```