source("renv/activate.R")
options(vsc.rstudioapi = TRUE) #added by `renvsc`
if (interactive() && Sys.getenv("TERM_PROGRAM") == "vscode") {
    source(file.path(Sys.getenv(if (.Platform$OS.type == "windows") "USERPROFILE" else "HOME"), ".vscode-R", "init.R"))
    # setup if using with vscode and R plugin
    options(vsc.rstudioapi = TRUE)
    # use the new httpgd plotting device
    options(vsc.plot = FALSE)
    options(device = function(...) {
      httpgd:::hgd()
      .vsc.browser(httpgd::hgd_url(), viewer = "Beside")
    })
}
