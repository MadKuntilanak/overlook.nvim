#  Disclaimer

This is just an additional feature related to my issue [#13](https://github.com/WilliamHsieh/overlook.nvim/issues/13)

I just want to mimic the behavior of the nvim-bqf window.

(WARN: The popup currently blocks the QF window when multiple QF windows are open 👻)

<details>
  <summary>Example configs</summary>

```lua
  {
    "MadKuntilanak/overlook.nvim",
    opts = {
      ui = {
        size_ratio = 0.60,
        min_width = 40,
        min_height = 20,
      },
      adapters = {
        peek_fzflua = {
          get = function(selection)
            local path = Utils.__strip_str(selection[1])
            if not path then
              return
            end

            local lnum = 1
            local col = 1

            if path:match ":" then
              path = vim.split(path, ":")

              local str_path = path[1]
              local _lnum = tonumber(path[2])
              local _col = tonumber(path[3])

              path = str_path

              if _lnum then
                lnum = _lnum
              end

              if _col then
                col = _col
              end
            end

            if path:match "~" then
              path = path:gsub("~", "")
              path = RUtils.config.path.home .. path
            end

            local bufnr = vim.fn.bufadd(path)
            if not vim.api.nvim_buf_is_loaded(bufnr) then
              vim.fn.bufload(bufnr)
            end

            local opts = {
              target_bufnr = bufnr,
              lnum = lnum,
              col = col,
              title = path,

              win_col = 50,
              win_row = 5,

              win_width = 70,
              win_height = 25,
            }

            return opts
          end,
        },
        peek_qf = {
          get = function()
            local lnum = vim.api.nvim_win_get_cursor(0)[1]
            local qflist = Utils.qf.get_list_qf_or_loc()
            local item = qflist[lnum]

            local opts = {}

            if item.bufnr then
              opts.target_bufnr = item.bufnr

              if item.text then
                if #item.text == 0 then
                  local path = vim.api.nvim_buf_get_name(item.bufnr)
                  opts.title = vim.fn.fnameescape(path)
                else
                  opts.title = item.text
                end
              end
            end

            if item.col then
              opts.col = item.col
            end

            if item.lnum then
              opts.lnum = item.lnum
            end

            opts.win_height = 20
            opts.win_width = 80

            opts.win_col = 1
            opts.win_row = -24

            return opts
          end,
        },
      },
    },
    keys = {
      {
        "P",
        function()
          local P = require "overlook.peek"
          P.peek_qf()
        end,
        ft = "qf",
        desc = "Peek item qf",
      },
      {
        "P",
        function()
          require("overlook.api").peek_definition()
        end,
        desc = "Peek definition",
      },
    },
  },
  ...
```

</details>

![Demo](./demo/abc.gif)
