# How to configure a static IP address.
* https://github.com/josenk/terraform-provider-esxi-wiki
## NOTES:
* Configuring your guest VM IP address is typically the scope of a provisioner... However, there is a lot of discussions in the issues section on this plugin, so I decided to take advice and put a wiki page.  If you have a solution you would like to share, please write up some details in Markdown and create a Pull Request.

* There are probably 100's of ways to set a static IP address for your guests.  There will be advantages and disadvantages to any solution.  Some will be simple and some will be complex.   Custom scripts tend to be propitiatory to a specific environment or OS.

## RULES:
* I DO NOT SUPPORT ANY OF THESE METHODS.  Don't post an issue in this gitrepo, I will close it.  Contact the author.
* I have full rights to accept or reject any PR.
* Only create Pull Requests for fully functional solutions that work with terraform-provider-esxi plugin.  This is not a general forum...
* I will not merge Questions, incomplete or broken solutions.
* If you are willing to support your solution (take questions and issues), you can put your email address in your PR.  If you don't want to support your solution, make sure your email address setting in github is not public.
* I will accept PRs to modify any solution (simple enhancements or typos only).  If the original author objects to those changes, I will revert to original.  The original author can request to completely modify or delete their document.
* Be in markdown format.  https://www.markdownguide.org/cheat-sheet/
* Each PR should contain:
  * Title:  How to configure a static IP address using < method >.
  * Description:  Write a short Description.
  * OS Support: List OS's that this method should work.
  * Pros:  List pros.
  * Cons:  List cons.
  * Details: Explain in detail how it's done. Use appropriate markdown for code, etc...
