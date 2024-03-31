
>参考文档：https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens

Personal access tokens（PCT） 是一种授权 GitHub 资源访问权限的方式。

目前，GitHub 支持2种 PCT：细粒度（fine-grained）PCT（GitHub 推荐）、经典（classic） PCT。2种PCT都与GitHub用户绑定，如果GitHub用户不能访问GitHub资源，那么通过该用户生成的PCT也无法访问GitHub资源。

---
下面介绍细粒度PCT（fgPCT）、经典PCT（cPCT）

fgPCT 的特点：
- fgPCT 只能访问用户、组织拥有的资源
- fgPCT 只能访问指定的仓库
- fgPCT 能够对访问权限做更加精细的控制
- fgPCT 有有效期（expiration date）
- Organization owners can require approval for any fine-grained personal access tokens that can access resources in the organization.

cPCT 的特点：
- cPCT 能够写公共仓库，生成该cPCT的用户该公共仓库的所有者，也非该公共仓库所有组织的成员之一。
- Outside collaborators can only use personal access tokens (classic) to access organization repositories that they are a collaborator on.
- 一些 REST API endpoints 只支持 cPCT，通过 `https://docs.github.com/en/rest/authentication/endpoints-available-for-fine-grained-personal-access-tokens` 查看 fgPCT 是否支持REST API endpoints。

使用 cPCT 意味着能够访问用户能访问的所有仓库（包含所在组织的所有仓库和用户的所有仓库），因此 GitHub 自动移除一年内没有使用的 cPCT，也建议用户为 cPCT 添加有效期。


# 创建 PCT





按照下面的步骤创建 cPCT。

）点击右上角头像，在弹出框中点击 "Settings"。

<img src="D:\NoteWithVersionControl\doc-git\8、Github\PIC\Pasted image 20240331005104.png" width=25%/>

）在左侧栏找到 "<> Developer settings"。

<img src="D:\NoteWithVersionControl\doc-git\8、Github\PIC\Pasted image 20240331005451.png" width=25%/>

）在左侧栏找到 "Token (classic)"。

<img src="D:\NoteWithVersionControl\doc-git\8、Github\PIC\Pasted image 20240331005601.png" width=25%/>

）点击 "Generate new token"，点击 "Generate new token (classic)"。

<img src="D:\NoteWithVersionControl\doc-git\8、Github\PIC\Pasted image 20240331005752.png" width=500/>
<img src="D:\NoteWithVersionControl\doc-git\8、Github\PIC\Pasted image 20240331005818.png" width=200/>

）在 Note 处填写Token描述信息，在 Expiration 处选择 Token 有效期。

<img src="D:\NoteWithVersionControl\doc-git\8、Github\PIC\Pasted image 20240331010602.png" width=60%/>

）选择 scope。（关于 scope 见 https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps）

）保存生成的 Token（注意该Token以后不会再出现）。 

<img src="D:\NoteWithVersionControl\doc-git\8、Github\PIC\Pasted image 20240331011003.png" width=70%/>



